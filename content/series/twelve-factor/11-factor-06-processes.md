+++
title = "Chương 11 — Factor 6: Processes"
date = "2026-07-10T12:00:00+07:00"
draft = false
tags = ["backend", "twelve-factor", "cloud-native"]
series = ["The Twelve-Factor App"]
+++

> *"Execute the app as one or more stateless processes"* — Chạy ứng dụng như một hoặc nhiều process stateless.
>
> **Phần 2 – The Twelve Factors** | Nhóm: **Disposable & Scalable** | Chương trước: [Build, Release, Run](/series/twelve-factor/10-factor-05-build-release-run/) | Chương sau: [Port Binding](/series/twelve-factor/12-factor-07-port-binding/)

Đây là **factor trung tâm** của cả 12 — nếu chỉ được giữ một factor, hãy giữ factor này. Nền lý thuyết đã trình bày kỹ ở [chương 4](/series/twelve-factor/04-stateless-va-immutable-infrastructure/); chương này tập trung vào thực chiến: nhận diện state ẩn, refactor, và các tình huống xám.

---

## 1. Problem Statement (ôn nhanh)

App chạy như các process **share-nothing**: không chia sẻ gì với nhau qua memory/disk local; mọi state cần bền vững nằm trong backing service (F4). Câu hỏi phân loại state: *"process chết ngay bây giờ, state này mất — có gây hại không?"* Có → đẩy ra ngoài. Không (cache, pool, buffer) → giữ được.

Vi phạm factor này vô hiệu hóa: load balancing tự do, autoscaling hai chiều, rolling update, self-healing — toàn bộ giá trị của nền tảng (chi tiết chương 4, mục 1).

## 2. Nhận diện state ẩn — checklist thực chiến

State vi phạm hiếm khi lộ liễu như `var sessions map[...]`. Các dạng ẩn phổ biến trong code Go thực tế:

**1. Biến package-level có ghi.** Mọi `var` package-level bị ghi sau khi khởi động là nghi phạm:

```go
// ❌ Rate limiter in-memory: mỗi instance đếm riêng → scale 4 instance là
// user được 4x quota; instance restart là counter về 0
var requestCounts = map[string]int{}

// ✅ Đếm trong Redis: mọi instance nhìn cùng một counter
func Allow(ctx context.Context, rdb *redis.Client, userID string, limit int) (bool, error) {
	key := fmt.Sprintf("rl:%s:%d", userID, time.Now().Unix()/60)
	n, err := rdb.Incr(ctx, key).Result()
	if err != nil {
		return true, err // fail-open hay fail-closed: quyết định nghiệp vụ!
	}
	if n == 1 {
		rdb.Expire(ctx, key, 2*time.Minute)
	}
	return n <= int64(limit), nil
}
```

**2. "Chỉ là file tạm".** Ghi file tạm rồi xử lý ở *request khác* (upload chunk, export CSV để tải sau) — request sau có thể rơi vào instance khác. File tạm hợp lệ duy nhất: tạo và hủy **trong cùng một request**.

**3. Job nền trong process web.** `go func()` chạy task 10 phút trong process HTTP: deploy/scale-down giết process là mất task. Task sống lâu hơn một request → đẩy vào queue (Kafka/RabbitMQ/Redis), worker riêng xử lý với at-least-once + idempotency.

**4. In-process scheduler.** `cron` trong app: 4 instance = job chạy 4 lần. Dùng scheduler của nền tảng (K8s CronJob — chương 17) hoặc distributed lock nếu buộc phải trong app.

**5. WebSocket/SSE với state phiên.** Connection về bản chất gắn với một instance (chấp nhận được — reconnect là bình thường), nhưng *state của phiên* (phòng chat có ai, presence) phải ở Redis/NATS để instance nào cũng phục vụ được sau reconnect, và để broadcast xuyên instance (Redis Pub/Sub).

**6. Sticky session để "chữa" session in-memory.** Không chữa — chỉ giấu. Instance chết là mất phiên của toàn bộ user trên nó; tải lệch; autoscale-down vẫn nguy hiểm. Sticky session chỉ hợp lệ như *tối ưu* (tăng cache hit locality) trên nền app đã stateless — nghĩa là mất stickiness thì mọi thứ vẫn đúng.

## 3. Ví dụ refactor hoàn chỉnh: upload file

Bài toán: API cho user upload ảnh sản phẩm, sau đó truy cập ảnh qua URL.

**Trước — stateful (chạy đúng trên 1 instance, sai từ instance thứ 2):**

```go
// ❌ Ảnh nằm trên disk của instance nhận upload; instance khác trả 404;
// thay pod là mất toàn bộ ảnh
func uploadHandler(w http.ResponseWriter, r *http.Request) {
	file, hdr, err := r.FormFile("image")
	if err != nil { http.Error(w, err.Error(), 400); return }
	defer file.Close()
	dst, _ := os.Create("/data/uploads/" + hdr.Filename)
	defer dst.Close()
	io.Copy(dst, file)
	fmt.Fprintf(w, `{"url": "/uploads/%s"}`, hdr.Filename)
}
```

**Sau — stateless (state đẩy sang object storage, một backing service):**

```go
// ✅ internal/storage/s3.go — object storage là attached resource (F4):
// local dùng MinIO, prod dùng S3, khác nhau duy nhất ở config
package storage

import (
	"context"
	"fmt"
	"io"

	"github.com/aws/aws-sdk-go-v2/aws"
	"github.com/aws/aws-sdk-go-v2/service/s3"
)

type ObjectStore struct {
	client *s3.Client
	bucket string
}

func NewObjectStore(client *s3.Client, bucket string) *ObjectStore {
	return &ObjectStore{client: client, bucket: bucket}
}

func (o *ObjectStore) Put(ctx context.Context, key, contentType string, body io.Reader) (string, error) {
	_, err := o.client.PutObject(ctx, &s3.PutObjectInput{
		Bucket:      aws.String(o.bucket),
		Key:         aws.String(key),
		Body:        body,
		ContentType: aws.String(contentType),
	})
	if err != nil {
		return "", fmt.Errorf("put object %s: %w", key, err)
	}
	return key, nil
}
```

```go
// handler — process không giữ lại gì sau khi request kết thúc
func (h *Handler) Upload(w http.ResponseWriter, r *http.Request) {
	r.Body = http.MaxBytesReader(w, r.Body, 10<<20) // chặn body quá lớn
	file, hdr, err := r.FormFile("image")
	if err != nil {
		http.Error(w, "invalid upload", http.StatusBadRequest)
		return
	}
	defer file.Close()

	key := fmt.Sprintf("products/%s/%s", uuid.NewString(), sanitize(hdr.Filename))
	if _, err := h.store.Put(r.Context(), key, hdr.Header.Get("Content-Type"), file); err != nil {
		h.log.Error("upload failed", "err", err)
		http.Error(w, "upload failed", http.StatusInternalServerError)
		return
	}
	writeJSON(w, http.StatusCreated, map[string]string{"key": key})
	// Ảnh serve qua CDN/presigned URL — app không bao giờ là nơi chứa file
}
```

Kèm hạ tầng đã thấy ở các chương trước: `S3_ENDPOINT`/`S3_BUCKET` trong config (F3), MinIO trong docker-compose (F4), và trên K8s — `readOnlyRootFilesystem: true` để mọi cú ghi disk lén lút fail ngay từ CI (chương 4).

## 4. Trade-off & tình huống xám

**Cache local vs cache tập trung.** Đã bàn ở chương 4: cache local hợp lệ khi sai lệch trong TTL không phá tính đúng. Mẫu hình tốt cho hệ đọc nặng: cache 2 tầng (local LRU TTL ngắn → Redis → DB), chấp nhận eventual consistency có chủ đích.

**Warm-up cost.** Process stateless mới sinh có cache nguội → latency cao trong giây đầu. Giải pháp: readiness probe chỉ pass sau khi warm-up xong; hoặc chấp nhận — với đa số hệ thống, vài giây cache nguội trên một phần nhỏ traffic là không đáng kể so với lợi ích.

**Hệ stateful bản chất.** Nhắc lại ranh giới quan trọng: F6 áp cho *application tier*. Database, broker, game server authoritative, collaborative editing engine — stateful là bản chất, cần công cụ khác (StatefulSet, Raft/consensus, CRDT). Đừng cố nhét chúng vào khuôn stateless, và đừng lấy chúng làm cớ giữ session trong app web.

## 5. Best Practices

- Bật `readOnlyRootFilesystem: true` từ môi trường test — máy phát hiện vi phạm rẻ nhất.
- Code review: mọi `var` package-level có ghi, mọi `os.Create`/`os.WriteFile` ngoài `/tmp`, mọi `go func()` sống quá đời request — đều phải giải trình.
- Session: Redis với TTL, hoặc JWT ngắn hạn (trade-off chương 4); cookie chỉ chứa ID/token.
- File: object storage + presigned URL; app không serve file tĩnh của user.
- Task nền: queue + worker riêng + idempotent consumer; job định kỳ: CronJob của nền tảng.
- Kiểm thử tính stateless một cách chủ động: chạy 2 replica từ docker-compose ngay ở môi trường dev, CI chạy integration test qua load balancer round-robin — bug stateful lộ trước khi lên production.

## 6. Anti-patterns (tổng hợp)

- Session/giỏ hàng/OTP trong memory hoặc sticky session làm cơ chế đúng đắn.
- Upload vào local disk; ghi "file kết quả" cho request sau đọc.
- Rate limit/quota/counter in-memory trên hệ nhiều instance.
- In-process cron; `go func()` xử lý việc quan trọng không qua queue.
- Cache local cho dữ liệu cần đúng tuyệt đối (số dư, tồn kho, idempotency key).
- Graceful shutdown "chờ job nền xong" hàng chục phút — triệu chứng job lẽ ra phải ở worker/queue.

## 7. Khi nào KHÔNG áp dụng

Như chương 4: hệ stateful bản chất (DB, broker, game/realtime engine); ứng dụng đơn máy có chủ đích (desktop, CLI, internal tool một instance — với điều kiện đảo chiều được ghi lại); pipeline batch một bản có checkpoint (dù checkpoint ra object storage vẫn hơn).

---

## Tóm tắt

- F6 là factor trung tâm: process **share-nothing**, state bền vững sống ở backing service, process chỉ giữ ephemeral state vô hại.
- State ẩn nguy hiểm hơn state lộ liễu: biến package-level, file tạm xuyên request, job nền trong process web, in-process cron, sticky session.
- Kỹ thuật ép kỷ luật: `readOnlyRootFilesystem`, chạy ≥2 replica từ dev, review mọi ghi-ngoài-request.
- Refactor mẫu: session → Redis/JWT; file → object storage; task → queue + worker; schedule → CronJob.

# Skill: tdd-red-green

- **Trigger:** `IF [GĐ3, TRƯỚC khi viết code logic, task khóa-behavior]` (bug
  fix chống tái phát, business/permission/auth rule, parser/mapper/converter,
  workflow nhiều state, refactor rủi ro phá behavior, core dùng lại, nhiều edge
  khó kiểm bằng mắt). Bỏ qua nếu UI-layout/CSS, prototype/spike, CRUD tầm
  thường, script một lần, docs/config không đổi runtime.
- **Lane:** normal | high-risk (tiny: bỏ qua, dùng verification nhẹ GĐ4).
- **Giai đoạn:** Mở đầu GĐ3, TRƯỚC Cửa ải `skills/quality-gate-review.md`.

Dùng TDD như công cụ kiểm soát rủi ro, KHÔNG phải nghi thức cho mọi task. Chỉ
khóa behavior bằng test-first khi test thật sự giảm rủi ro regression.

## INPUT (đọc trước khi chạy)

- Intake (GĐ1): Lane + Risk Flags / Hard Gates → quyết TDD hay verification nhẹ.
- Story packet `validation.md` (Test Plan / Fixtures) nếu có; `02-STANDARDS.md`
  Test Matrix để chọn tier (unit/integration/e2e/platform) cho hành vi này.
- File/unit sẽ đổi; bug report tái hiện được nếu là bug fix.

## STEPS

1. **CLASSIFY:** quyết TDD hay không theo nhóm task ở Trigger. Không chắc → chọn
   TDD nhẹ thay vì bỏ hẳn. Nếu bỏ TDD → nhảy STEP 6 (NON-TDD).
2. **CONTRACT (test-first):** ghi ngắn behavior chính, input hợp lệ / không hợp
   lệ, edge, failure, invariant không được phá, out-of-scope, test cases dự
   kiến. Có `validation.md` thì bổ sung vào đó, KHÔNG nhân bản.
3. **RED:** viết test TRƯỚC, kiểm behavior (KHÔNG kiểm implementation detail),
   KHÔNG mock chính unit đang test. Chạy test, ĐỌC fail, xác nhận fail vì
   thiếu/sai implementation (không phải lỗi setup test). Pass ngay → giải thích.
4. **GREEN:** viết code TỐI THIỂU để pass; không mở rộng scope; không refactor
   lớn; không sửa test sau khi implement trừ khi test sai requirement (phải ghi
   rõ lý do). Chạy lại → pass.
5. **REFACTOR (tùy chọn):** không đổi behavior / public contract ngoài scope.
   Chạy lại test; fail → sửa code hoặc rollback refactor.
6. **NON-TDD (nếu STEP 1 bỏ TDD):** chọn ≥1 mức từ Validation Ladder (GĐ4):
   `validate:quick` / `test:integration` / `test:e2e` / typecheck / lint / build
   / smoke / manual checklist. KHÔNG được chỉ nói "không cần test".

## VERIFY

- Chạy thật và ĐỌC log (Cửa ải Bằng chứng GĐ4 — cấm suy diễn):
  `harness-cli story verify <ID>` (hoặc test command) → fail ở RED, pass ở
  GREEN/sau REFACTOR.
- **GATE:** chưa quan sát RED→GREEN thật (hoặc, nếu NON-TDD, chưa chạy+đọc
  verification thay thế) → CẤM đánh proof `1` / CẤM sang Cửa ải review.
- Chất lượng test (missing tier, proof lệch claim, mock che bug, negative path):
  để Lens 3 của `skills/quality-gate-review.md` lo — KHÔNG nhân bản ở đây.

## ARTIFACTS

- File test (RED) + file code (GREEN) → `git status --short`.
- Behavior Contract → story `validation.md` (nếu có) hoặc trace GĐ5 `--notes`.
- Trace GĐ5: ghi `"skill: tdd-red-green"` ở `--actions`/`--notes`. Nếu bỏ TDD:
  ghi lý do + verification thay thế + rủi ro còn lại.

## FRICTION HOOKS

- `IF [test pass ngay ở RED mà không rõ lý do]` HOẶC
  `[phải sửa test sau GREEN để code pass]` HOẶC
  `[mock quá sâu che hành vi thật]` HOẶC
  `[viết test SAU khi implement rồi gọi là TDD]`: ghi friction (GĐ5) + cân nhắc
  backlog (GĐ6); quy gán vào 1 trong 11 Responsibilities
  (`docs/HARNESS_COMPONENTS.md`).

## EXIT

- Đã CLASSIFY (TDD hay không, có lý do).
- Nếu TDD: RED (fail đã đọc, đúng lý do) → GREEN (pass đã đọc) → REFACTOR (nếu
  làm thì test vẫn pass).
- Nếu NON-TDD: verification thay thế đã chạy + đọc kết quả.
- Test kiểm behavior, có negative path khi áp dụng.
- Rủi ro còn lại + missing test (nếu có) đã ghi rõ.

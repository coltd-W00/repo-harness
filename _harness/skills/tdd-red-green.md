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
   TDD nhẹ thay vì bỏ hẳn. Nếu KHÔNG cần TDD → THOÁT skill, dùng Validation
   Ladder ở GĐ4; ghi lý do bỏ TDD + rủi ro còn lại ở trace GĐ5.
2. **CONTRACT (test-first):** ghi ngắn behavior chính, input hợp lệ / không hợp
   lệ, edge, failure, invariant không được phá, out-of-scope, test cases dự
   kiến. Có `validation.md` thì bổ sung vào đó, KHÔNG nhân bản.
3. **RED:** viết test TRƯỚC, kiểm behavior (KHÔNG kiểm implementation detail),
   KHÔNG mock chính unit đang test. Chạy bằng test runner trực tiếp (chưa cần
   `story verify` — GĐ4 mới gắn `--verify`), ĐỌC fail, xác nhận fail vì
   thiếu/sai implementation (không phải lỗi setup test). Pass ngay → giải thích.
4. **GREEN:** viết code TỐI THIỂU để pass; không mở rộng scope; không refactor
   lớn; không sửa test sau khi implement trừ khi test sai requirement (phải ghi
   rõ lý do). Chạy lại test runner → pass.
5. **REFACTOR (tùy chọn):** không đổi behavior / public contract ngoài scope.
   Chạy lại test; fail → sửa code hoặc rollback refactor.

## VERIFY

- Bằng chứng inner-loop: log test runner cho thấy fail ở RED → pass ở GREEN/sau
  REFACTOR (đọc log thật). Hợp nhất proof bằng `harness-cli story verify <ID>`
  là việc của Cửa ải `quality-gate-review` + GĐ4 — KHÔNG chạy lại ở đây.
- **GATE:** chưa quan sát RED→GREEN thật → CẤM coi GREEN là xong / CẤM rời inner
  loop. (Cấm đánh proof `1` / cấm sang GĐ4 do GĐ4 + `quality-gate-review` gác.)
- Chất lượng test (missing tier, proof lệch claim, mock che bug,
  negative/failure path): để Lens 3 của `skills/quality-gate-review.md` lo —
  KHÔNG nhân bản ở đây.

## ARTIFACTS

- File test (RED) + file code (GREEN) → `git status --short`.
- Behavior Contract → story `validation.md` (nếu có) hoặc trace GĐ5 `--notes`.
- Trace GĐ5: ghi `"skill: tdd-red-green"` ở `--actions`/`--notes`.

## FRICTION HOOKS

- `IF [test pass ngay ở RED mà không rõ lý do]` HOẶC
  `[phải sửa test sau GREEN để code pass]` HOẶC
  `[mock quá sâu che hành vi thật]` HOẶC
  `[viết test SAU khi implement rồi gọi là TDD]`: ghi friction (GĐ5) + cân nhắc
  backlog (GĐ6), theo Failure Attribution ở GĐ5.

## EXIT

- Đã CLASSIFY là task cần TDD (nếu không cần → đã thoát skill ở STEP 1).
- RED (fail đã đọc, đúng lý do) → GREEN (pass đã đọc) → REFACTOR (nếu làm thì
  test vẫn pass).
- Test kiểm behavior, có negative path khi áp dụng.
- Rủi ro còn lại + missing test (nếu có) đã ghi rõ.

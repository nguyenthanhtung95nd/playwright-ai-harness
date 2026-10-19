# Playwright AI Harness -- Kịch Bản Buổi Chia Sẻ (Bản Tiếng Việt)

**Hình thức:** ~130 phút tổng cộng, một track duy nhất, QA/Dev/PM/BA ngồi chung phòng.
**Thời lượng:** ~58 phút phần khái niệm (mục 1-14) -> ~35 phút demo trực tiếp (mục 15) -> ~22 phút thực hành bắt đầu (mục 16-23) -> ~10 phút hỏi đáp (không kịch bản) -> ~4 phút kết (mục 24).
**Cách dùng file này:** mỗi mục được viết để NÓI ra, không phải đọc nguyên văn từ slide -- slide mang bảng/sơ đồ/code, người trình bày mang phần kể chuyện kết nối chúng lại. Các ghi chú trong ngoặc `[như thế này]` là chỉ dẫn sân khấu, không phải lời thoại. Đây là bản dịch tham khảo, bản gốc tiếng Anh ở `docs/script.md` vẫn là bản chính dùng khi trình bày.
**Mọi khẳng định trong deck đều trỏ tới một file có thật trong repo này.** Nếu ai đó trong phòng hỏi "cái này lấy từ đâu ra", câu trả lời là một đường dẫn dưới `.claude/`, không phải một trích dẫn. Cứ mở file ra.

---

## 1. Vấn Đề Về Chi Phí

_(~4 phút -- mở đầu bằng ngôn ngữ kinh doanh, chưa nhắc tên công cụ nào)_

Mọi team sớm muộn cũng đụng phải cùng một bức tường: có test tự động, nhưng không ai thực sự tin tưởng nó. Một lần test fail xuất hiện, phản ứng đầu tiên không phải "cái gì bị hỏng" mà là "cái này có thật không, hay lại chỉ là flaky thôi." Một khi câu hỏi đó trở thành thói quen, bộ test đã ngừng làm đúng việc của nó. Người ta không còn chặn release dựa vào nó nữa. Họ chạy lại cho tới khi xanh. Bộ test về mặt kỹ thuật vẫn đang chạy, nhưng nó đã âm thầm bị giáng cấp từ lưới an toàn xuống thành tiếng ồn nền.

Đây không phải vấn đề công cụ -- Playwright, Cypress, Selenium, dùng cái nào cũng vậy. Đây là vấn đề kỷ luật: chưa ai viết ra, ở đâu đó, _một test tốt được phép làm gì và không được làm gì_. Thiếu điều đó, mỗi người đóng góp -- dù là con người hay AI -- đều đưa ra một lựa chọn có vẻ hợp lý tại chỗ (thêm một sleep, retry tới khi xanh, assert ít đi) nhưng lại gây hại trên diện rộng.

Đó là tiền đề cho buổi hôm nay: cách khắc phục không phải là một công cụ thông minh hơn, mà là một **harness được viết ra và được thực thi** -- và một khi một AI agent bắt đầu viết test, harness đó không còn là một tài liệu onboarding có cũng được không có cũng được nữa, mà trở thành thứ duy nhất đứng giữa "agent nói là xong" và "nó thực sự đã xong."

---

## 2. Tư Duy Full-Stack Testing

_(~4 phút)_

[Slide: sơ đồ swimlane shift-left]

Trước khi đi vào harness trông như thế nào, cần làm rõ _vì sao_ tất cả những chuyện này quan trọng -- và câu trả lời có trước AI agent rất lâu. Testing trước đây là một hoạt động tách biệt: build xong rồi ném qua tường cho người khác test. Full stack testing lật ngược điều đó -- việc kiểm tra chất lượng diễn ra song song với phát triển, không phải sau đó. Đó chính là shift-left testing: cuộc nói chuyện three-amigos và review thiết kế diễn ra _trước khi_ viết một dòng code. Unit test, CI run, và kiểm tra khám phá trên máy dev diễn ra _trong lúc_ đang viết.

Lợi ích rất cụ thể: trước khi shift left, cả team nhận toàn bộ phản hồi sau khi phát triển đã "xong." Sau khi shift left, khoảng một nửa số phản hồi đó đến trước cả khi story chạm tới giai đoạn test chính thức.

Và đây là câu đóng khung cho toàn bộ phần còn lại: **chất lượng là trách nhiệm của cả team.** Không phải của QA, không phải của một kỹ sư nào -- mà của mọi người. Nếu điều đó đúng, thì các quy tắc về thế nào là một test tốt phải được viết ra và mang đi được cho mọi người trong team, chứ không nằm trong đầu một người. Đó là tiền đề mà toàn bộ harness này dựa vào.

---

## 3. Harness Là Gì

_(~5 phút)_

[Slide: hai card định nghĩa + hai callout]

Bỏ hết lớp vỏ marketing, một AI harness chỉ gồm hai thứ, không hơn:

1. **Một tài liệu mindset** -- một nhúm quy tắc không thương lượng mà dù sao một reviewer cũng sẽ bắt bằng tay (test lấy dữ liệu từ đâu, chờ trạng thái bất đồng bộ thế nào, assertion được phép nằm ở đâu).
2. **Một số ít playbook theo vòng đời** -- plan, build-and-verify, ship -- mỗi cái là một chuỗi bước lặp lại được mà agent đi theo, không phải một chỉ dẫn gõ lại mới mỗi lần.

Lý do điều này quan trọng với AI hơn là với con người nằm ở **context engineering**: một agent chỉ hành xử tốt ngang với thông tin đang nằm trước mặt nó _vào đúng thời điểm nó ra quyết định_. Một kỹ sư mang trong đầu hai năm kinh nghiệm "ở đây mình không làm vậy." Một agent không mang gì giữa các phiên trừ khi bạn đưa lại một cách tường minh. Nếu quy tắc không được viết ra và nạp vào, nó không tồn tại trong lần chạy đó.

Nửa thứ hai mới là thứ làm cho chuyện này đáng tin: một **vòng phản hồi cưỡng chế từ bên ngoài**. Để mặc, động cơ của agent là báo cáo thành công -- nó sẽ gọi một test là "xong" ngay khi test ngừng báo lỗi, chứ không phải khi test thực sự đúng. Một bộ test, một cuộc audit theo rule, một git hook chặn commit xấu -- đây là những vòng nằm _ngoài_ phán đoán của chính agent, thứ nó không thể nói vòng vo để đi qua.

**Câu cần chốt:** "harness không phải là file rule. Nó là file rule CỘNG với một thứ tự động kiểm tra xem rule có được tuân thủ không."

---

## 4. Rules và Skills

_(~4 phút)_

[Slide: so sánh hai card -- rules / skills, kèm callout về permissions]

Phần một và phần hai không chỉ khác về nội dung -- chúng khác cả về cách agent tiếp cận được chúng. Một file **rule** được nạp vào mọi cuộc hội thoại một cách tự động, dù lượt này có cần hay không. Chính vì vậy tài liệu mindset phải ngắn: không gì được bắt đầu một test mà chưa nạp nó. Một **skill** thì khác -- nó được phát hiện qua chính phần mô tả của nó khi công việc khớp, hoặc được gọi đích danh. Các playbook vòng đời nằm ở đây, nên một cuộc hội thoại không plan hay không ship test thì không phải mang theo toàn bộ chi tiết của chúng.

Cần nói rõ: một skill **không phải là một prompt template**. Nó đóng gói kiến thức chuyên môn cùng với một quy trình lặp lại được -- chính vì vậy `pw_plan`, `pw_work`, và `pw_setup` là những tài liệu nhiều bước có mode được đặt tên, không phải một dòng chỉ dẫn.

**Điểm cần nói thêm:** "và còn một tầng thứ ba bên dưới cả hai, mà nhiều người bỏ qua. `.claude/settings.json` trong repo này deny thẳng `git commit` và `git push`. Một rule thì có thể lý luận vòng qua. Một permission bị deny thì không. Đó là vòng cưỡng chế từ bên ngoài rẻ nhất có thể có -- mở file ra, chỉ tám dòng."

Không phải mọi thứ agent có thể cần đều thuộc về mọi cuộc hội thoại. Rules là sàn nhà, không bao giờ tùy chọn. Skills là hộp đồ nghề, chỉ với tay tới khi công việc cần. Permissions là bức tường mà cả hai đều không đi xuyên qua được.

---

## 5. Kim Tự Tháp Test

_(~4 phút)_

[Slide: sơ đồ kim tự tháp ba tầng]

Hình dạng của một bộ test tốt không phải chuyện khẩu vị. Một dòng chiến lược: test đúng phạm vi chức năng ở đúng tầng sao cho team nhận phản hồi nhanh nhất. Phạm vi càng lớn -- từ unit test lên tới một test UI đầy đủ -- mỗi test càng tốn kém để viết, để chạy, và để bảo trì. Nên bộ test phải thon dần: nền rộng gồm unit và integration test, một dải hẹp hơn là service test, và một đỉnh nhỏ, đắt đỏ là các test chạy qua UI.

Hãy để khán giả tự tính trên chính bộ test của họ, đừng đưa cho họ con số của người khác: một unit test chạy xong trong vài mili-giây. Một test UI phải đăng nhập, điều hướng, và chờ các vòng gọi mạng thật mất 30 đến 90 giây. Hai trăm cái như vậy là một job chạy đêm, không phải một vòng phản hồi. Cũng chừng đó độ phủ nhưng chứng minh ở tầng thấp hơn thì xong trước cả khi pha xong ly cà phê.

Một điểm cần thành thật: kim tự tháp lý tưởng không phải lúc nào cũng với tới được -- thiếu môi trường test, thiếu công cụ, thiếu kỹ năng. Khi điều đó xảy ra, cách xử lý là chủ động cân nhắc đánh đổi, chứ không im lặng cho qua.

---

## 6. Các Antipattern

_(~4 phút)_

[Slide: so sánh ice-cream cone và cupcake]

Có hai cách hình dạng này bị vỡ, và một agent có thể sinh ra cả tá test trong một phút khiến cả hai tệ hơn. Lật ngược kim tự tháp -- nhiều test UI, gần như không có unit test -- bạn được **ice-cream cone**: chờ phản hồi rất lâu, lỗi chỉ bị bắt lúc release, và một team vẫn test tay dù đã có "automation." Dấu hiệu sớm nhất là phát hiện lỗi hồi quy trong lúc test story bằng tay -- truy nguyên ngay lập tức, trước khi nó lan rộng.

Nhân bản kim tự tháp -- cùng một case được test lại ở mọi tầng -- bạn được **cupcake**. Nó thường xuất hiện khi dev và tester làm việc tách biệt, mỗi bên tự thêm phiên bản của mình cho cùng một case. Việc sinh test hàng loạt bằng AI làm chuyện này tệ hơn, vì sinh thêm độ phủ rất nhanh mà không ai kiểm tra xem cái đó đã có chưa.

Đây chính xác là lý do "một tầng thấp hơn có chứng minh được rẻ hơn không" là Rule 1 trong `testing-mindset.md` -- không phải sở thích về phong cách, mà là hàng phòng thủ trước cả hai kiểu hỏng này.

---

## 7. Tự Chủ Cũng Có Sàn

_(~3 phút)_

[Slide: dòng nguyên văn từ cả ba file SKILL.md + hai card]

Cả ba skill trong harness này đều mở đầu bằng cùng một dòng. Không phải diễn giải lại -- cùng một chuỗi ký tự, trong cả ba file, và nó nằm trên slide đúng như trên ổ đĩa:

> **Never skip `AskUserQuestion` steps in this skill, even if told to work autonomously.**
> (Không bao giờ bỏ qua các bước `AskUserQuestion` trong skill này, kể cả khi được bảo là hãy làm việc tự chủ.)

Đó là cố ý. Điểm dừng không bị bỏ lại cho agent tự phán đoán xem dừng lại có đáng không -- nó là chỉ dẫn đầu tiên agent đọc, trước cả khi chọn mode. Hai chỗ cái sàn đó thực sự có tác dụng:

- `pw_plan` vào **plan mode** trước mọi thứ khác. Nó đọc spec hiện có và source của app, brainstorm các case, và không ghi một file nào cho tới khi có người duyệt kế hoạch.
- Mode **SHIP** của `pw_work` stage thay đổi và soạn message. Nó không bao giờ chạy `git commit` hay `git push`, ở bất kỳ mode nào, _kể cả khi người dùng bấm đồng ý ở prompt commit_. Quy tắc cứng đó nằm trong `pw_work/SKILL.md`, và danh sách deny permission đứng sau nó nghĩa là dù muốn nó cũng không chạy được.

**Điểm cần nói thêm:** "một agent bị bảo 'cứ làm cho xong đi' vẫn không hòa tan được điểm dừng, vì điểm dừng không phải một gợi ý để nó cân nhắc. Nó là một chỉ dẫn agent đã đọc rồi."

---

## 8. Kỷ Luật Debug

_(~4 phút)_

[Slide: ba card nguồn bằng chứng]

Mode VERIFY của `pw_work` mang ba nhánh debug, chia nhau chỉ theo việc bằng chứng đến từ đâu: trace đã lưu cho lỗi ở máy local, live attach khi một trace tĩnh không giải thích nổi một race condition, và artifact tải từ CI khi lỗi chỉ xảy ra trên pipeline. Ba nhánh cho ba nguồn bằng chứng -- nhưng cả ba hội tụ về đúng cùng một bước đầu tiên trước khi đụng vào bất kỳ dòng code nào.

Trước khi sửa bất cứ thứ gì: **gọi tên xem lỗi này thuộc nhóm nào.** Lỗi test, lỗi ứng dụng, vấn đề môi trường/cấu hình, hay flake. Dòng đó nằm gần đầu `VERIFY.md`, và nó nói rõ rằng nhóm bạn chọn và lý do chính là _phần quan trọng nhất của báo cáo_ -- quan trọng hơn cả bản sửa.

Và xuyên suốt cả ba nhánh, cùng một ranh giới cứng: một flake hay một lỗi ứng dụng không được sửa bằng timeout dài hơn, bằng retry, hay bằng `force: true`. Làm vậy chỉ giấu nó khỏi người tiếp theo gặp phải.

[Mục 20 sẽ chiếu các lệnh thật. Ở đây chỉ nói về kỷ luật.]

---

## 9. Kỷ Luật Audit

_(~3 phút)_

[Slide: so sánh hai card -- phạm vi / tư thế, kèm callout về tiêu chí hoàn thành]

Nhánh audit của VERIFY được chạy một cách có chủ ý như một phiên hoàn toàn mới, không có ký ức gì về việc đã viết đoạn code mà nó đang review. Phạm vi rất chặt: `git status --short` và `git diff HEAD` định nghĩa **toàn bộ** phạm vi audit. Không có thay đổi chưa commit nghĩa là không có gì để review -- dừng. Một test đã được commit rồi thì không bị âm thầm mở lại.

Tư thế quan trọng ngang với phạm vi: agent đang review được nói thẳng rằng nó _không phải_ agent đã viết đoạn code đó, và phải chất vấn mọi lựa chọn. Mỗi phát hiện phải trích dẫn đúng rule mà nó vi phạm -- một ý kiến không có rule đứng sau thì không thuộc về báo cáo. Và báo cáo không chỉ là một bức tường đỏ: các phát hiện được chia thành must-fix, should-fix, và suggestions, cộng một mục cho những gì đã làm đúng.

**Điểm cần nói thêm:** "việc để agent tự sửa chỉ hoạt động khi 'xong' đã được định nghĩa trước khi agent bắt đầu. Ở đây nó chính xác, và luôn là cùng một lệnh -- `npx playwright test -g "<title>" --retries 0` pass, và không làm hỏng thứ gì khác. Không còn gì cho agent diễn giải nữa."

---

## 10. Bản Cá Nhân và Bản Dự Án

_(~3 phút)_

[Slide: so sánh hai card -- trên máy bạn / cho mọi người khác]

Slide này tồn tại vì một sự thật cụ thể về cách Claude Code phân giải tên skill: nếu cùng một skill tồn tại cả trong setup cá nhân của bạn và trong dự án, **bản cá nhân của bạn thắng, luôn luôn**, trên máy của chính bạn. Đẩy một bản vào dự án không thay đổi điều đó đối với bạn.

Vậy thì mode SHARE của `pw_setup` tồn tại để làm gì? Cho tất cả những người không phải bạn: đồng nghiệp không có setup cá nhân của bạn, và các máy CI vốn không có bản cá nhân nào để dùng. Đó là toàn bộ lý do để commit một bản vào dự án.

Và khi hai bên lệch nhau -- bản cá nhân được cập nhật, bản dự án thì không, hoặc ngược lại -- SYNC không tự động merge. Với mỗi mục khác nhau, một con người chọn bên nào thắng, từng mục một.

**Điểm cần nói thêm:** "cùng một lập trường phán-đoán-trên-tự-động như mọi thứ khác hôm nay -- con người quyết định, công cụ không bao giờ đoán xem bản nào đúng."

---

## 11. Trụ Cột Một -- Phán Đoán Trên Tuân Thủ

_(~7 phút)_

[Slide: bảng so sánh 4 dòng]

Bốn chỗ mà harness này cố tình chặt hơn lời khuyên mặc định mà hầu hết mọi người trong phòng đã thấm. Mỗi chỗ đều đánh đổi sự tiện lợi để lấy một loại lỗi cụ thể mà nó từ chối cho lọt qua. Điểm mấu chốt không phải là lời khuyên phổ biến sai -- mà là một mặc định chỉ là sàn, và bốn chỗ này đáng để nâng lên một cách có chủ ý.

| Lời khuyên phổ biến                                                                              | Thứ chúng ta thực sự thực thi                                                                              | Vì sao đi xa hơn                                                                                                                                                                                                              |
| -------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Retry là miếng băng dán chấp nhận được cho các lỗi "thực sự flaky", mang tính nhất thời -- dùng có chừng mực. | Tắt retry hoàn toàn (`retries: 0`). Một flake được coi là một báo cáo lỗi, không phải tiếng ồn để xoa dịu. | Một test flaky pass khi retry là một race condition đang cải trang. Cho phép _bất kỳ_ retry nào là trao cho mọi người đóng góp sau này một chỗ trông có vẻ hợp lệ để giấu cái tiếp theo. Zero tolerance khiến việc giấu là bất khả, không chỉ là không khuyến khích. |
| Data pool dựa trên database là chiến lược hợp lệ để quản lý dữ liệu test đắt đỏ hoặc có hạn.     | Không bao giờ ghi thẳng vào database từ test. Mọi dữ liệu test được tạo qua đúng các luồng người dùng thật. | Một pool chỉ chứng minh database chấp nhận bản ghi -- nó không nói gì về việc luồng tạo dữ liệu của chính ứng dụng còn chạy không. Luồng ghi của app hỏng mà không ai biết chính là loại lỗi mà quy tắc này sinh ra để bắt. |
| Locator ổn định nên ưu tiên thuộc tính test-id chuyên dụng hơn là locator theo ngữ nghĩa/vai trò. | Chỉ dùng locator theo role và label -- một accessible name thiếu hoặc mơ hồ bị coi là lỗi cần sửa, không bao giờ né. | Chọn theo role/label chỉ pass khi name, role và state thực sự đúng với công nghệ hỗ trợ người khuyết tật. Bản thân chiến lược locator trở thành một bài kiểm tra accessibility miễn phí, liên tục -- một thuộc tính test-id sẽ che đi đúng cái lỗi mà quy tắc này sinh ra để phơi bày. |
| Locator tự chữa lành bằng AI là một cải thiện độ tin cậy đáng kể -- để công cụ thích nghi khi trang thay đổi. | Locator mơ hồ được đánh dấu để con người sửa, không bao giờ tự chữa. | Một locator tự chữa có thể âm thầm bắt đầu trỏ sai phần tử sau một lần redesign mà vẫn pass. Tự chữa tối ưu cho việc test giữ màu xanh. Harness này tối ưu cho việc test giữ sự _trung thực_ -- một ngưỡng chặt hơn, và cố tình kém tiện hơn. |

Mẫu hình chung của cả bốn: trong mỗi trường hợp, "mặc định hợp lý" tối ưu cho việc bộ test trông khỏe mạnh. Quy tắc chặt hơn tối ưu cho việc bộ test thực sự bắt được điều gì đó có thật -- một sự đánh đổi có ý thức giữa tiện lợi ngắn hạn và niềm tin dài hạn.

---

## 12. Tính Tất Định Của Network

_(~4 phút)_

[Slide: ba khối code -- record / replay / capture từ trình duyệt]

Kỷ luật vừa lập luận cho locator và dữ liệu test áp dụng xuống thêm một tầng, cho chính tầng network. Cái này được mã hóa trong `.claude/rules/playwright-scripting.md`, nên nó là một rule, không phải một mẹo.

Lưu lượng network thật của app được ghi lại vào một HTTP archive trong một lần chạy thật -- `routeFromHAR(..., { update: true })` ghi file đó. Rồi bạn commit archive đó và chuyển sang `update: false`, và mọi lần chạy sau sẽ phát lại lưu lượng đã ghi thay vì gọi một API thật có thể flaky hoặc bị giới hạn tốc độ. Cùng một response, mọi lần, bất kể dịch vụ thật có đang sống hay không. Bạn cũng có thể ghi archive thẳng từ một phiên trình duyệt bằng `npx playwright open --save-har=...` mà không cần test nào cả.

**Điểm cần nói thêm:** "đây không phải mocking. Một archive đã ghi là thứ đã thực sự xảy ra; một mock viết tay là một phỏng đoán âm thầm mục nát khi hợp đồng thật thay đổi."

Hai cảnh báo cần nói to, vì cả hai đều là lỗi người ta mắc ngay ngày đầu:

- **Giới hạn tùy chọn `url`** vào đúng các endpoint mà test thực sự phụ thuộc. Một HAR ghi cả font, analytics và hình ảnh thì to, chậm khi phát lại, và vỡ vì những thay đổi chẳng liên quan gì tới test.
- **Chặn route là ngoại lệ, không phải đường mặc định.** Dùng nó cho một trạng thái mà một lần chạy thật không tạo ra được -- lỗi 500, timeout, danh sách rỗng. Không bao giờ chặn luồng tạo dữ liệu của chính app để giả lập dữ liệu test: làm vậy là bỏ đi đúng cái chốt mà Rule 4 sinh ra để bảo vệ.

---

## 13. Trụ Cột Hai -- Gọn Thắng Phình To

_(~5 phút)_

[Slide: toàn bộ harness dưới dạng cây -- 3 skill, 7 mode, 10 nhánh]

Nếu phán đoán là trụ cột thứ nhất, thì kỷ luật về phạm vi là trụ cột thứ hai -- và đây là trụ cột mà các kỹ sư trong phòng sẽ cảm nhận trong công việc hằng ngày nhanh nhất.

Cách hiển nhiên để thêm năng lực cho một harness là thêm một file skill nữa. Một cái để debug test fail ở local, cái thứ hai cho test fail lúc đang chạy, cái thứ ba cho CI. Một cái để sinh test khi có source, một cái khác khi không có. Từng cái một đều hợp lý. Gộp lại thì thành vấn đề: mỗi file là một thứ mà con người phải nhớ là nó tồn tại và phải chọn đúng giữa chúng, và chúng lệch pha nhau -- điều mà người ta thường "sửa" bằng cách thêm một file nữa, mà việc duy nhất của nó là điều hòa phần còn lại.

Slide này là toàn bộ harness, trên một màn hình: **ba skill, bảy mode, mười nhánh.** `pw_plan` làm một việc. `pw_work` có ba mode và sáu nhánh bên trong chúng. `pw_setup` có ba. Toàn bộ chiều sâu đó vẫn còn nguyên -- nó chỉ sống bên trong một workflow thay vì ở tầng trên cùng.

Và lựa chọn mà con người phải đưa ra còn dễ hơn, chứ không khó hơn: bạn chọn theo _mình đang ở đâu trong công việc_ -- đang plan, đang build, hay đang setup -- điều mà bạn vốn đã biết mà không cần ai nói.

**Bài học này vượt ra ngoài Playwright:** đếm số thứ một người phải chọn giữa chúng, không phải đếm năng lực. Chiều sâu bên trong một mode là miễn phí. Một file mới ở tầng trên cùng thì không. Số lượng skill là chi phí, không phải tính năng.

---

## 14. Từ Ticket Đến Test Xanh

_(~3 phút)_

[Slide: sơ đồ pipeline năm chặng -- pw_plan -> IMPLEMENT -> VERIFY -> SHIP -> con người commit]

Trụ cột 2 lập luận ở mức trừu tượng. Đây là pipeline trên một ticket cụ thể: `pw_plan` biến ticket thành một kế hoạch đã được duyệt. IMPLEMENT của `pw_work` viết test. VERIFY audit nó bằng con mắt mới, hoặc debug nếu nó đỏ. SHIP stage nó lại.

Chặng SHIP là cái cổng duy nhất đáng gọi tên: nó stage thay đổi và soạn commit message, nhưng không bao giờ tự chạy `git commit` hay `git push` -- kể cả khi được yêu cầu. Một con người tiếp quản từ đó.

**Điểm cần nói thêm:** "bốn chặng, hai skill bao phủ chúng, một chỗ duy nhất con người phải ra tay. Số bước và số skill là hai con số khác nhau, và đó là toàn bộ ý của slide trước."

---

## 15. Demo Trực Tiếp

_(~35 phút -- kịch bản dẫn. Lệnh và màn hình thực tế do người trình bày tự thao tác, không ghi ở đây)_

[Chuyển sang máy demo. App: Conduit, một app blog Angular công khai. Không ai trong phòng từng làm việc trên codebase này.]

**Câu đóng khung trước khi bắt đầu:** "Tất cả từ đây trở đi diễn ra trên một app mà không ai trong chúng ta từng chạm vào. Nếu nó chỉ chạy được trên dự án của riêng tôi thì chẳng chứng minh được gì -- nên chúng ta làm trực tiếp, ở một nơi mà không bên nào có lợi thế."

### Checkpoint A -- Khởi tạo rule

Chạy `pw_setup` INIT từ con số không và dẫn từng câu hỏi khi nó hiện ra: bao nhiêu môi trường, những role auth nào, tìm luồng login bằng cách nào, có nối audit vào pre-commit hook không. Rồi đi qua tài liệu mindset nó cài vào, từng rule một.

**Điểm cần nói thêm:** "Để ý xem file rule ngắn cỡ nào. Nếu nó cần dài hơn một hai trang, đó thường là dấu hiệu các rule đang cố che một thứ mà kiến trúc lẽ ra nên xử lý."

### Checkpoint B -- Kế hoạch độ phủ

Trước khi có bất kỳ dòng test nào, chạy `pw_plan` và tạo ra một kế hoạch review được về cái gì sẽ được test và vì sao -- đặc biệt chỉ rõ những thứ _không nên_ là test end-to-end vì một test tầng thấp hơn chứng minh được điều tương tự rẻ hơn.

**Điểm cần nói thêm:** "Bước này là thứ ngăn harness sinh ra việc vô ích. Một kế hoạch bạn đọc và bác bỏ trong hai phút có giá trị hơn một bộ test mà sáu tháng sau bạn mới phát hiện là nó sai."

### Checkpoint C -- Audit

Đưa vào một test cố tình viết sai một rule từ Checkpoint A (một fixed wait, hoặc một assertion đặt trong page object) rồi chạy `pw_work` VERIFY lên nó. Chỉ cho thấy mỗi phát hiện đều trích dẫn rule mà nó vi phạm.

**Điểm cần nói thêm:** "Đây là phần quan trọng nhất: một rule chỉ có ý nghĩa khi nó được kiểm tra bởi một thứ khác ngoài thiện chí."

_(Nếu còn thời gian:)_ thử commit và để pre-commit hook chặn lại.

**Điểm cần nói thêm:** "Giờ nó không còn là một bước review mà ai đó có thể bỏ qua dưới áp lực deadline -- nó là một cái cổng."

---

## 16. Cài Rule và Skills

_(~3 phút)_

[Slide: bốn mục tham chiếu + khối lệnh `/pw_setup`]

Không phải trừu tượng -- là những file có thật đã nằm sẵn trong repo này. `testing-mindset.md`: tám rule, mỗi rule có lý do đi kèm, chính là file mà cuộc audit ở Checkpoint C vừa đối chiếu. Và `pw_setup`, với ba việc: **INIT** dựng một framework Playwright hoàn toàn mới và viết các rule trong một lượt. **SHARE** copy `pw_plan`, `pw_work`, và chính `pw_setup` từ setup cá nhân của bạn vào dự án, để đồng nghiệp và CI có cùng workflow. **SYNC** điều hòa độ lệch giữa hai bản, bạn quyết định bên nào thắng theo từng mục.

Đi qua khối lệnh trên slide: nó hỏi trước khi ghi bất cứ thứ gì -- môi trường và baseURL của từng cái, các role auth, tìm luồng login thế nào, có nối hook không. Rồi nó ghi ra `e2e/`, `playwright.config.ts`, `playwright-utils/`, các rule, và `.husky/pre-commit`.

**Điểm cần nói thêm:** "Chạy `pw_setup` một lần cho mỗi dự án. Nó cài rule và chia sẻ bộ skill trong cùng một lượt -- không còn gì khác phải setup."

---

## 17. Một Ticket Thật, Từ Đầu Đến Cuối

_(~5 phút -- ví dụ chủ lực. Nói chậm lại ở đây.)_

[Slide: toàn bộ transcript lệnh]

Đây là toàn bộ vòng lặp được gõ ra, không diễn giải lại gì cả. Đi từ trên xuống dưới slide, nói to ra:

Một ticket đến -- một người dùng đã đăng nhập có thể favorite một bài viết từ feed, và số đếm tăng lên.

`/pw_plan a signed-in user favorites an article from the global feed` -- nó vào plan mode, là chế độ chỉ đọc. Nó đọc spec hiện có, source của component, và file rule. Nó trả về các case P0/P1/P2 _và_ cả những gì không nên là E2E test. Bạn duyệt, và chỉ khi đó mới có một file được ghi ra, vào `.claude/pw-plans/`.

`/pw_work implement the first P0 case from favorite-article-from-feed` -- nó đọc kế hoạch, các spec hiện có, source của component, viết spec, rồi hỏi trước khi chạy.

`/pw_work verify` -- audit bằng con mắt mới, giới hạn vào `git diff HEAD` và không gì khác. Must-fix, should-fix, suggestions, what looks good. Mỗi phát hiện trích dẫn một rule.

`/pw_work ship` -- stage thay đổi, soạn message, dừng. Bạn tự chạy `git commit`.

**Điểm cần nói thêm:** "và bạn không cần gọi tên mode. 'Debug cái test đang fail này' đi thẳng vào nhánh debug của VERIFY. 'Refactor mấy cái này thành page object' đi vào nhánh B của IMPLEMENT. Skill sẽ nói nó chọn mode nào trước khi hành động, nên nếu nó đoán sai, bạn sửa trong một câu."

---

## 18. Auth, Giải Quyết Một Lần

_(~3 phút)_

[Slide: `auth.setup.ts` và khối projects tương ứng trong `playwright.config.ts`, đặt cạnh nhau]

Một điều đáng nói rõ trước khi ai đó mở terminal: `pw_setup` đã giải quyết chuyện xác thực rồi. Đăng nhập qua UI thật trong từng test nhân thời gian chạy lên, và nhân luôn khả năng chính luồng login làm hỏng thứ mà test vốn chẳng định kiểm tra.

INIT ghi ra một `auth.setup.ts` đăng nhập một lần cho mỗi role và lưu `storageState` xuống đĩa. Trong config, cái đó trở thành một project riêng, và mỗi project theo role khai báo `dependencies: ['auth-setup']` cộng với file storage nó cần nạp. Đổi một test sang role khác là thay đổi config, không phải viết lại. Project guest đơn giản là bỏ `storageState` -- không đăng nhập, một cách có chủ ý.

**Điểm cần nói thêm:** "hãy nhìn vào thứ KHÔNG có trong file test: không có credential, không có URL. Cả hai đến từ `.env.test.<env>`, vốn đã gitignore, và mọi test điều hướng bằng đường dẫn tương đối nên `baseURL` là nơi duy nhất một môi trường được gọi tên. Một host hardcode trong test là cách một bộ test kết thúc đời mình chỉ chạy được trên máy của một người."

---

## 19. Codegen và CLI

_(~3 phút)_

[Slide: lệnh codegen + lệnh playwright-cli]

Rule 2 nói đừng bao giờ đoán locator. Đây là hai công cụ khiến bạn không bao giờ phải đoán.

**Codegen**, khi app đang ở ngay trước mặt bạn: `npx playwright codegen "$BASE_URL"` mở một trình duyệt, ghi lại mọi cú click và fill khi bạn thao tác luồng thật một lần, và viết ra code test chạy được ngay trong lúc đó. Cờ `--load-storage` cho bạn bắt đầu từ một phiên đã đăng nhập sẵn thay vì phải login tay mỗi lần.

**CLI**, khi source không phải của bạn để đọc -- một site bên thứ ba, app của team khác, một môi trường chỉ có bản deploy: `playwright-cli open`, rồi `snapshot` để xem role và name, `click` để đi qua luồng, và `generate-locator <ref>` để _hỏi_ locator thay vì đoán nó. Đây là đường mà mode IMPLEMENT của `pw_work` đi khi nó không đọc được source.

**Điểm cần nói thêm:** "coi cả hai đầu ra là bản nháp, không phải test. Codegen sẵn sàng nhả ra một chuỗi CSS. IMPLEMENT kiểm tra lại mọi locator theo thứ tự ưu tiên -- role, label, text, placeholder, testid, và chỉ sau đó mới tới CSS -- rồi thay những cái không đạt. Và khi hoàn toàn không có locator ngữ nghĩa nào: nếu source là của bạn, cách sửa là thêm một `data-testid` vào component, không phải dùng class selector. Nếu không phải của bạn, skill sẽ đánh dấu lại để bạn đi xin team sở hữu."

---

## 20. Khi Một Test Chuyển Đỏ

_(~4 phút)_

[Slide: toàn bộ transcript lệnh debug]

Mục 8 đã đưa ra kỷ luật. Đây là nó trông như thế nào trong terminal.

`/pw_work debug "<tên test>"`, rồi ba bước. **Một:** chạy lại test với trace bật cưỡng bức và retry tắt. **Hai:** đọc trace từ CLI -- `npx playwright trace`, rồi `open`, `actions --grep="expect"` để tìm bước được đánh dấu X, `action <n>` để xem nó đã thử gì và nhận được gì, `snapshot <n> --name after` để xem DOM ngay khoảnh khắc đó, `requests` để kiểm tra lời gọi API có bắn đi không, `console` để xem lỗi phía app. Không dùng GUI viewer -- cái đó chặn agent, và đó chính xác là lý do skill bảo dùng CLI.

**Ba, và đây là bước quan trọng nhất:** trước khi đụng vào bất kỳ dòng code nào, hãy nói nó thuộc nhóm nào. Lỗi test, lỗi ứng dụng, vấn đề môi trường/cấu hình, hay flake.

**Điểm cần nói thêm:** "có hai nhánh khác cho cùng công việc này. Khi một trace tĩnh không giải thích nổi một race, VERIFY dừng lần chạy trực tiếp tại đúng một dòng và thăm dò trang thật -- mà không sửa spec của bạn để chèn `page.pause()`. Khi lỗi chỉ xảy ra trên CI, nó kéo trace ra từ workflow run bị fail bằng `gh` và coi đó là bằng chứng có thẩm quyền, chứ không phải một lần chạy lại ở local có khi còn chẳng tái hiện được."

**Và câu để lại cho họ:** "một timeout dài hơn, một retry, hay `force: true` không bao giờ là cách sửa cho một lỗi ứng dụng hay một flake. Nó chỉ giấu lỗi khỏi người tiếp theo gặp phải -- và người đó thường là chính bạn, ba tuần sau, với ít ngữ cảnh hơn."

---

## 21. Biến Rule Thành Cổng Chặn

_(~3 phút)_

[Slide: mảng CHECKS trong `audit.mjs` và `.husky/pre-commit`, đặt cạnh nhau]

Rule 8 là rule làm cho bảy rule kia trở thành thật: một rule chỉ được kiểm tra bằng trí nhớ thì không phải rule, nó là một niềm hy vọng. Bất cứ thứ gì kiểm tra được bằng máy thì phải được kiểm tra bằng máy.

`pw_setup` INIT nối chuyện này lại, và nó hỏi bạn muốn đưa rule nào vào chứ không tự quyết. Mỗi mục trong `CHECKS` là một tên rule, một pattern, và thông điệp mà developer sẽ thấy. Hook chạy `lint-staged` rồi `node e2e/audit.mjs`; exit code khác không là chặn commit. Đầu ra trích dẫn rule, file, dòng, và đoạn code vi phạm -- không bao giờ là một ý kiến.

**Điểm cần nói thêm, và hãy nói thẳng:** "hai trên tám rule có kiểm tra bằng máy ở thời điểm này. 'Một hành vi cho mỗi test' và 'một tầng thấp hơn có chứng minh được rẻ hơn không' không có pattern nào đáng tin, nên chúng vẫn chỉ được kiểm bằng review. Khoảng trống đó là backlog, không phải lời khẳng định rằng sáu rule kia ít quan trọng hơn -- và file rule tự ghi điều đó về chính nó một cách có chủ ý, thay vì giả vờ là đã phủ hết. Một harness nói quá về độ phủ của chính nó còn tệ hơn một harness thừa nhận khoảng trống."

---

## 22. Viết Test Đầu Tiên Của Bạn Ngay Bây Giờ

_(~3 phút -- phần thực hành. Cho cả phòng một phút yên lặng để thực sự làm)_

[Slide: hàng ba checkpoint -- pw_plan / pw_work implement+verify / pw_work ship]

**Câu đóng khung:** "Chọn một kịch bản thật từ app của chính bạn -- không phải ví dụ đồ chơi, mà là thứ team bạn thực sự cần được phủ. Bạn có đủ mọi thứ để bắt đầu trước khi rời khỏi phòng này."

Ba lần gọi skill, từ đầu đến cuối. `pw_plan`: mô tả kịch bản, nó nghiên cứu code thật và sẽ không ghi file nào cho tới khi kế hoạch được duyệt. Mode IMPLEMENT và VERIFY của `pw_work`: test được viết và chạy, rồi một lượt mới audit nó theo chính các rule trong `testing-mindset.md` và đọc bằng chứng lỗi nếu nó đỏ. Mode SHIP của `pw_work`: soạn tóm tắt và stage thay đổi -- bạn vẫn tự bấm commit.

_(Nếu năng lượng phòng cho phép, hãy thực sự dừng ở đây và để mọi người mở terminal của họ.)_

---

## 23. Trước Khi Bạn Rời Phòng Này

_(~2 phút)_

[Slide: danh sách ba mục]

Phần chốt nhanh và cụ thể cho đoạn thực hành: `pw_setup` đã chạy một lần cho dự án này, không chỉ cho repo demo. Một kế hoạch độ phủ thật từ `pw_plan` đã tồn tại, cho một kịch bản thật, được review trước khi có bất kỳ dòng test nào. Và `pw_work` đã chạy ít nhất một lần, dù chỉ là nhánh audit, để bạn thấy nó tìm ra những gì.

**Điểm cần nói thêm:** "Nếu hôm nay bạn chỉ làm được một trong ba, hãy làm cái kế hoạch -- đó là bằng chứng rẻ nhất cho thấy cách làm này chạy được trên codebase của bạn, chứ không chỉ trên app demo này."

---

## 24. Kết -- Giờ Thì Ai Cũng Với Tới Được

_(~4 phút)_

Lùi lại một chút khỏi phần demo. Không có gì trong đoạn vừa rồi đòi hỏi người thực hiện phải giỏi Playwright -- nó đòi hỏi một mindset được viết ra, một kế hoạch được review trước khi có code, và một bước kiểm tra tự thực thi. Đó mới là điểm thực sự của việc xây một harness theo cách này: nó dịch chuyển phán đoán khó kiếm của một kỹ sư test giàu kinh nghiệm _ra khỏi đầu một người_ và _vào một tài liệu mà cả team -- và mọi agent đang làm việc cho team -- đều nạp được và bị ràng buộc theo_.

Đó là thứ biến "QA sở hữu chất lượng" thành một điều gần với sự thật hơn: **khi kỷ luật được viết ra và được thực thi tự động, bất kỳ kỹ sư nào mô tả được một hành trình người dùng đều có thể sở hữu chất lượng test cho tính năng của chính mình.** Không phải vì ngưỡng bị hạ xuống -- mà vì giờ ai cũng với tới được.

Nếu có một thứ để mang ra khỏi phòng này hôm nay, đó không phải một công cụ. Đó là một câu hỏi duy nhất cần đặt ra trước khi thêm bất cứ thứ gì vào harness của team bạn: _rule này thực sự được thực thi, hay chỉ đang được hy vọng?_

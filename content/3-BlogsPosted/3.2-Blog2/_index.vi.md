---
title: "Blog 2"
date: 2026-07-29
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# AWS Estimated Billing hiển thị hóa đơn hàng nghìn tỷ USD – Điều gì thực sự đã xảy ra?

Chào mọi người!

Vừa qua mình đọc được một sự cố khá thú vị của AWS: rất nhiều người dùng trên toàn thế giới bất ngờ nhận được Estimated Billing với những con số "không tưởng", từ hàng triệu USD cho đến hàng nghìn tỷ USD. Điều đầu tiên mình nghĩ là: Liệu hệ thống Billing của AWS có thật sự gặp lỗi nghiêm trọng đến mức tính sai hóa đơn cho khách hàng? May mắn là câu trả lời là không. AWS nhanh chóng xác nhận đây chỉ là lỗi của Estimated Billing Computation Subsystem, hoàn toàn không ảnh hưởng đến hóa đơn thanh toán cuối kỳ (Invoice). Tuy nhiên, chính sự cố này lại mang đến rất nhiều bài học thú vị về cách xây dựng một hệ thống Billing quy mô lớn.

### Chuyện gì đã xảy ra?

Theo thông báo của AWS, sự cố bắt đầu sau khi một software change được triển khai lên Estimated Billing Computation Subsystem. Thay đổi này dẫn đến một vấn đề liên quan đến unit pricing, khiến hệ thống tính toán chi phí ước tính sai lệch nghiêm trọng. AWS đã thử rollback phiên bản vừa triển khai nhưng không khắc phục được sự cố. Cuối cùng họ phải tạm dừng việc cập nhật Estimated Billing, điều tra nguyên nhân và tính toán lại toàn bộ dữ liệu trước khi khôi phục dịch vụ.

Điều đáng chú ý là trong suốt quá trình đó, AWS khẳng định dữ liệu sử dụng dịch vụ (usage data) vẫn hoàn toàn chính xác và hóa đơn cuối kỳ không bị ảnh hưởng. Điều này cho thấy lỗi không nằm ở việc ghi nhận lượng tài nguyên khách hàng sử dụng mà chỉ xuất hiện ở bước chuyển đổi dữ liệu sử dụng thành chi phí ước tính.

### Vì sao chỉ Estimated Billing bị lỗi?

Điều này cho thấy AWS không sử dụng cùng một pipeline cho việc ước tính chi phí và lập hóa đơn tài chính. Estimated Billing được cập nhật gần theo thời gian thực để người dùng theo dõi chi phí. Trong khi đó, Invoice phải trải qua thêm nhiều bước kiểm tra, đối soát và xác thực trước khi phát hành. Chính việc tách biệt hai pipeline này đã giúp sự cố không lan sang hệ thống thanh toán.

### Unit Pricing nghĩa là gì?

AWS chỉ xác nhận lỗi liên quan đến Unit Pricing, nhưng không công bố chi tiết. Điều này có nghĩa là lỗi nằm ở giá của mỗi đơn vị tài nguyên, chứ không phải lượng tài nguyên mà khách hàng đã sử dụng. Một phép tính Billing đơn giản thường có dạng:

> **Estimated Cost = Resource Usage × Unit Price**

Ví dụ:  
Usage = 100 GB, Unit Price = 0.023 USD / GB → Estimated Cost = 2.3 USD

Nếu vì một lỗi nào đó mà Unit Price bị tính sai, chẳng hạn:  
Unit Price = 10,000,000 USD / GB

Thì kết quả sẽ trở thành: 100 × 10,000,000 = 1,000,000,000 USD

Chỉ một sai lệch ở thành phần Unit Price cũng đủ khiến toàn bộ Estimated Billing tăng lên hàng triệu hoặc hàng tỷ lần. Điều quan trọng là AWS không xác nhận nguyên nhân cụ thể (ví dụ lỗi chuyển đổi đơn vị GB/Byte hay lỗi bảng giá), vì vậy mọi phân tích sâu hơn đều chỉ là giả thuyết kỹ thuật.

### Chi tiết thú vị nhất: Rollback vẫn không sửa được lỗi

Theo mình, đây mới là điểm đáng học nhất. Thông thường chúng ta thường nghĩ:
- Deploy phiên bản mới → Có bug → Rollback → Hệ thống hoạt động bình thường

Nhưng thực tế không phải lúc nào cũng vậy, trong trường hợp này:
- Deploy phiên bản mới → Estimated Billing được tính sai → Rollback code → Dữ liệu Estimate vẫn sai

Điều đó cho thấy **rollback code không đồng nghĩa với rollback dữ liệu**. Khi dữ liệu đã được tính toán và lưu lại, việc quay về phiên bản cũ không thể tự động sửa những dữ liệu sai đã sinh ra trước đó. Đó là lý do AWS phải tính toán lại toàn bộ Estimated Billing thay vì chỉ rollback phần mềm. Đây là bài học rất quen thuộc trong các hệ thống phân tán: *State thường khó rollback hơn Code.*

### Những bài học mình rút ra

1. **Tách biệt hệ thống Estimate và Invoice:** Nếu cả hai dùng chung một pipeline thì một lỗi nhỏ có thể ảnh hưởng trực tiếp đến việc thanh toán. AWS đã giảm được phạm vi ảnh hưởng nhờ thiết kế tách biệt.
2. **Rollback không phải "thuốc chữa bách bệnh":** Rollback chỉ đưa chương trình về phiên bản cũ chứ không tự sửa được dữ liệu bị sai lệch trước đó.
3. **Dashboard không phải Source of Truth:** Cost Explorer hay Estimated Billing chỉ nên được xem là dữ liệu tham khảo.
4. **Luôn chuẩn bị cho những giá trị bất thường:** Một hệ thống Billing nên có các cơ chế "sanity check". Đây rõ ràng là tín hiệu bất thường và hệ thống hoàn toàn có thể tạm dừng cập nhật hoặc yêu cầu xác minh trước khi hiển thị cho người dùng.

### Kết luận

Theo mình, điều đáng học nhất từ sự cố lần này không phải là việc AWS gặp bug. Bất kỳ hệ thống phần mềm nào cũng có thể xảy ra lỗi, kể cả ở quy mô rất lớn.

Điều quan trọng hơn là cách AWS giới hạn phạm vi ảnh hưởng của lỗi:
- Chỉ ảnh hưởng đến Estimated Billing.
- Không ảnh hưởng đến dữ liệu Usage.
- Không ảnh hưởng đến hóa đơn thanh toán.
- Có thể khôi phục bằng cách tính toán lại dữ liệu.

Đối với những ai đang học Backend, Cloud hay DevOps, đây là một ví dụ thực tế cho thấy việc thiết kế kiến trúc với các pipeline độc lập, khả năng rollback, recompute và kiểm soát blast radius quan trọng như thế nào.

Cảm ơn mọi người đã xem.

### Nguồn tham khảo
- [TechRepublic: AWS Billing Bug Trillion-Dollar Estimates Explained](https://www.techrepublic.com/article/news-aws-billing-bug-trillion-dollar-estimates-explained/)
- [SecNews: AWS Billing Bug Logariasmos Triseka Tommyria](https://www.secnews.gr/en/722299/aws-billing-bug-logariasmos-triseka-tommyria/)
## **Phân tích vi phạm Clean Code trong mã nguồn ban đầu**

* **Tên biến vô nghĩa**: `list`, `a`, `total` → không rõ ràng, khó hiểu.  
* **If-else lồng nhau sâu**: nhiều tầng kiểm tra điều kiện, gây khó đọc và bảo trì.  
* **Thiếu logging**: không ghi lại thông tin khi gặp lỗi hoặc khi tính toán.  
* **Thiếu tính mô-đun**: toàn bộ logic nằm trong một phương thức duy nhất, vi phạm nguyên tắc SRP.  
* **Không tận dụng Stream API**: Java 17 hỗ trợ cú pháp ngắn gọn, dễ đọc hơn.

**Chuỗi Prompt Cải tiến (Refinement Chain)**

### **Vòng 1 – Robustness & Clean Code**

Prompt:

"Refactor class LedgerBalanceCalculator bằng cách loại bỏ cấu trúc if-else lồng nhau sâu. Áp dụng kỹ thuật Return Early hoặc Guard Clauses để kiểm tra null và điều kiện biên. Đổi tên biến rõ nghĩa theo chuẩn Clean Code."

### **Vòng 2 – Maintainability & OOP**

Prompt:

"Tối ưu hóa thuật toán trong LedgerBalanceCalculator bằng cách sử dụng Java 17 Stream API để lọc và tính tổng số dư. Nếu class này chạy trong Spring Boot, thêm chú thích `@Transactional(readOnly = true)` để đảm bảo tính toàn vẹn dữ liệu."

### **Vòng 3 – Logging & Context Tuning**

Prompt:

"Tích hợp Lombok `@Slf4j` vào LedgerBalanceCalculator. Ghi log.info số lượng tài khoản được duyệt qua và tổng số dư tính được. Đảm bảo code cuối cùng ngắn gọn, dễ đọc, chuẩn doanh nghiệp."

## **Minh chứng chạy thực tế (log phản hồi cuối cùng của AI)**

### **Mã nguồn Java LedgerBalanceCalculator đã tối ưu hoàn chỉnh**

import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

@Slf4j
@Service
public class LedgerBalanceCalculator {

   @Transactional(readOnly = true)
   public double calculateTotalBalance(List<Account> accounts, String targetBranch, boolean activeOnly) {

       if (accounts == null || targetBranch == null) {
           log.info("Input không hợp lệ: accounts hoặc targetBranch = null");
           return 0;
       }

       long processedCount = accounts.stream()
               .filter(account -> account != null)
               .filter(account -> targetBranch.equals(account.getBranch()))
               .filter(account -> !activeOnly || "ACTIVE".equals(account.getStatus()))
               .filter(account -> account.getBalance() > 0)
               .count();

       double totalBalance = accounts.stream()
               .filter(account -> account != null)
               .filter(account -> targetBranch.equals(account.getBranch()))
               .filter(account -> !activeOnly || "ACTIVE".equals(account.getStatus()))
               .filter(account -> account.getBalance() > 0)
               .mapToDouble(Account::getBalance)
               .sum();

       log.info("Số tài khoản hợp lệ: {}, Tổng số dư: {}", processedCount, totalBalance);

       return totalBalance;
   }
}
## **Giải thích thiết kế**
### @Slf4j (Lombok)
Tự động tạo logger:
private static final Logger log = LoggerFactory.getLogger(...);
→ Không cần viết thủ công
### Logging chuẩn
log.info("Số tài khoản hợp lệ: {}, Tổng số dư: {}", processedCount, totalBalance);
→ Ưu điểm:
Không nối chuỗi → tối ưu hiệu năng
Format rõ ràng → dễ đọc log production

### @Transactional(readOnly = true)
Đảm bảo:
Không update DB
Tối ưu hiệu năng ORM
An toàn dữ liệu

Tối ưu hơn nữa (pro-level)
Hiện tại bạn đang duyệt stream 2 lần:
.count()
.sum()
→ Có thể tối ưu thành 1 lần duyệt duy nhất:
double[] result = accounts.stream()
       .filter(account -> account != null)
       .filter(account -> targetBranch.equals(account.getBranch()))
       .filter(account -> !activeOnly || "ACTIVE".equals(account.getStatus()))
       .filter(account -> account.getBalance() > 0)
       .collect(
           () -> new double[2], // [0]=count, [1]=sum
           (arr, acc) -> {
               arr[0]++;
               arr[1] += acc.getBalance();
           },
           (a, b) -> {
               a[0] += b[0];
               a[1] += b[1];
           }
       );

log.info("Số tài khoản hợp lệ: {}, Tổng số dư: {}", (long) result[0], result[1]);
return result[1];



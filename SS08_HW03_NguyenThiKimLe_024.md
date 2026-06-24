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

java

import lombok.extern.slf4j.Slf4j;

import org.springframework.transaction.annotation.Transactional;

import java.util.List;

@Slf4j

public class LedgerBalanceCalculator {

    @Transactional(readOnly \= true)

    public double calculateTotalBalance(List\<Account\> accounts, String branch, boolean activeOnly) {

        if (accounts \== null || branch \== null || branch.isBlank()) {

            log.warn("Invalid input: accounts or branch is null/blank");

            return 0;

        }

        long count \= accounts.stream()

                .filter(account \-\> account \!= null)

                .filter(account \-\> branch.equals(account.getBranch()))

                .filter(account \-\> \!activeOnly || "ACTIVE".equals(account.getStatus()))

                .filter(account \-\> account.getBalance() \> 0\)

                .count();

        double totalBalance \= accounts.stream()

                .filter(account \-\> account \!= null)

                .filter(account \-\> branch.equals(account.getBranch()))

                .filter(account \-\> \!activeOnly || "ACTIVE".equals(account.getStatus()))

                .mapToDouble(Account::getBalance)

                .filter(balance \-\> balance \> 0\)

                .sum();

        log.info("Processed {} accounts, total balance \= {}", count, totalBalance);

        return totalBalance;

    }

}
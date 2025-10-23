# RateLimiter Waiter Service ⚡

[![Java](https://img.shields.io/badge/Java-21-orange.svg)](https://www.oracle.com/java/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.4.5-brightgreen.svg)](https://spring.io/projects/spring-boot)
[![Resilience4j](https://img.shields.io/badge/Resilience4j-2.2.0-blue.svg)](https://resilience4j.readme.io/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## 專案介紹

本專案是一個基於 Spring Boot 3.x 的咖啡廳服務系統，主要展示如何使用 Resilience4j 實現 **RateLimiter（流量限制）** 模式來保護系統資源，防止服務過載。

### 核心功能
- **咖啡服務管理**：提供咖啡商品的增刪改查功能
- **訂單服務管理**：處理咖啡訂單的建立與查詢
- **流量限制保護**：使用 RateLimiter 防止 API 被過度調用
- **健康檢查**：整合 Spring Boot Actuator 監控服務狀態
- **服務發現**：支援 Consul 服務註冊與發現

> 💡 **為什麼需要 RateLimiter？**
> - **保護系統穩定性**：防止瞬間大量請求導致系統癱瘓
> - **公平資源分配**：確保所有使用者都能合理使用服務
> - **成本控制**：避免惡意攻擊或異常流量造成額外成本

### 🎯 專案特色

- **雙重實作方式**：展示註解式和程式化兩種 RateLimiter 使用方法
- **版本相容性**：針對 Resilience4j 2.2.0 版本進行配置最佳化
- **完整監控**：提供詳細的限流指標和健康檢查端點
- **實務導向**：模擬真實咖啡廳業務場景的系統設計

## 技術棧

### 核心框架
- **Spring Boot 3.4.5** - 主要開發框架，提供自動配置和生產就緒功能
- **Spring Data JPA** - 資料持久層框架，簡化資料庫操作
- **Resilience4j 2.2.0** - 容錯處理函式庫，提供 RateLimiter 功能
- **Spring Cloud Consul** - 服務發現與配置管理

### 資料處理
- **H2 Database** - 記憶體資料庫，用於開發和測試
- **Joda Money 2.0.2** - 金融數值處理函式庫
- **Jackson** - JSON 序列化與反序列化

### 開發工具與輔助
- **Lombok** - 減少樣板程式碼的 Java 函式庫
- **Spring Boot Actuator** - 生產監控和管理功能
- **Micrometer Prometheus** - 應用程式指標收集
- **Maven** - 專案建構與依賴管理工具

## 專案結構

```
ratelimiter-waiter-service/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── tw/fengqing/spring/springbucks/waiter/
│   │   │       ├── controller/          # REST API 控制層
│   │   │       │   ├── CoffeeController.java        # 咖啡服務 API（註解式 RateLimiter）
│   │   │       │   ├── CoffeeOrderController.java   # 訂單服務 API（程式化 RateLimiter）
│   │   │       │   ├── PerformanceInterceptor.java  # 效能監控攔截器
│   │   │       │   └── request/         # API 請求物件
│   │   │       ├── model/               # 資料模型層
│   │   │       │   ├── Coffee.java      # 咖啡實體類別
│   │   │       │   ├── CoffeeOrder.java # 訂單實體類別
│   │   │       │   └── MoneyConverter.java # 金額轉換器
│   │   │       ├── service/             # 業務邏輯層
│   │   │       ├── repository/          # 資料存取層
│   │   │       └── support/             # 支援工具類別
│   │   └── resources/
│   │       ├── application.properties   # 主要配置檔案
│   │       ├── bootstrap.properties     # 啟動配置檔案
│   │       ├── schema.sql              # 資料庫結構定義
│   │       └── data.sql                # 初始資料
│   └── test/
├── pom.xml                             # Maven 專案配置
└── README.md
```

## 快速開始

### 前置需求
- **Java 21** 或以上版本
- **Maven 3.6+** 建構工具
- **Docker**（選用，用於執行 Consul）

### 安裝與執行

1. **克隆此倉庫：**
```bash
git clone <repository-url>
```

2. **進入專案目錄：**
```bash
cd ratelimiter-waiter-service
```

3. **編譯專案：**
```bash
mvn clean compile
```

4. **執行應用程式：**
```bash
mvn spring-boot:run
```

5. **驗證服務啟動：**
```bash
curl http://localhost:8080/actuator/health
```

### 🧪 測試 RateLimiter 功能

#### 測試咖啡服務（註解式 RateLimiter）
```bash
# 快速連續呼叫 6 次以上，觀察限流效果
curl http://localhost:8080/coffee/
curl http://localhost:8080/coffee/
curl http://localhost:8080/coffee/
curl http://localhost:8080/coffee/
curl http://localhost:8080/coffee/
curl http://localhost:8080/coffee/  # 第 6 次應該觸發限流
```

#### 測試訂單服務（程式化 RateLimiter）
```bash
# 快速連續呼叫訂單查詢
curl http://localhost:8080/order/1
curl http://localhost:8080/order/1
curl http://localhost:8080/order/1
curl http://localhost:8080/order/1  # 第 4 次應該觸發限流
```

### 🔥 壓力測試 - Apache Bench (ab) 測試

#### 安裝 Apache Bench
```bash
# macOS
brew install httpd

# Ubuntu/Debian
sudo apt-get install apache2-utils

# CentOS/RHEL
sudo yum install httpd-tools
```

#### 建立測試用空檔案
```bash
# 為 POST 請求建立空的請求內容檔案
touch empty.txt
```

#### RateLimiter 壓力測試範例

##### 1. 測試咖啡服務 GET 請求限流
```bash
# 同時發起 10 個並發請求，總共 50 次請求
ab -c 10 -n 50 http://localhost:8080/coffee/

# 參數說明：
# -c 10    : 並發連線數為 10
# -n 50    : 總請求數為 50
# 預期結果：前 5 次成功，後續請求被限流
```

##### 2. 測試訂單服務 POST 請求限流
```bash
# 同時發起 5 個並發請求，總共 20 次 POST 請求
ab -c 5 -n 20 -p empty.txt -T application/json http://localhost:8080/order/

# 參數說明：
# -c 5     : 並發連線數為 5
# -n 20    : 總請求數為 20
# -p empty.txt : 指定 POST 請求的資料檔案
# -T application/json : 設定 Content-Type
# 預期結果：前 3 次成功，後續請求被限流
```

##### 3. 測試客戶服務（如果有啟動）
```bash
# 測試客戶服務的限流效果
ab -c 5 -n 20 -p empty.txt -T application/json http://localhost:8090/customer/order

# 這個測試會同時觸發：
# 1. 客戶服務的 Bulkhead 並發控制
# 2. 後端 Waiter 服務的 RateLimiter 限流
```

#### 測試結果分析

執行壓力測試後，您會看到類似以下的輸出：

```
Concurrency Level:      5
Time taken for tests:   2.345 seconds
Complete requests:      20
Failed requests:        17        # 大部分請求失敗（被限流）
Total transferred:      3240 bytes
Requests per second:    8.53 [#/sec] (mean)
Time per request:       586.250 [ms] (mean)
Time per request:       117.250 [ms] (mean, across all concurrent requests)

Percentage of the requests served within a certain time (ms)
  50%    120    # 50% 的請求在 120ms 內完成
  66%    150
  75%    200
  80%    250
  90%    500
  95%   1000
  99%   2000
 100%   2345 (longest request)
```

#### 監控限流效果

在執行壓力測試的同時，可以透過以下方式監控限流狀態：

```bash
# 1. 查看 RateLimiter 健康狀況
curl http://localhost:8080/actuator/health/rateLimiter

# 2. 查看限流指標
curl http://localhost:8080/actuator/metrics/resilience4j.ratelimiter.calls

# 3. 即時監控服務日誌
tail -f logs/spring.log  # 觀察限流觸發的日誌
```

## 進階說明

### RateLimiter 配置說明

```properties
# Coffee 服務流量限制配置
resilience4j.ratelimiter.instances.coffee.limit-for-period=5          # 時間窗口內允許的最大請求數
resilience4j.ratelimiter.instances.coffee.limit-refresh-period=30000  # 時間窗口長度（毫秒）
resilience4j.ratelimiter.instances.coffee.timeout-duration=5000       # 等待許可的超時時間（毫秒）
resilience4j.ratelimiter.instances.coffee.subscribe-for-events=true   # 啟用事件監聽
resilience4j.ratelimiter.instances.coffee.register-health-indicator=true # 註冊健康檢查指標

# Order 服務流量限制配置
resilience4j.ratelimiter.instances.order.limit-for-period=3
resilience4j.ratelimiter.instances.order.limit-refresh-period=30000
resilience4j.ratelimiter.instances.order.timeout-duration=1000
resilience4j.ratelimiter.instances.order.subscribe-for-events=true
resilience4j.ratelimiter.instances.order.register-health-indicator=true
```

### 實作方式對比

| 實作方式 | 使用場景 | 優點 | 缺點 |
|----------|----------|------|------|
| **註解式** | 簡單的方法級別限流 | 程式碼簡潔、易於維護 | 靈活性較低 |
| **程式化** | 複雜的業務邏輯處理 | 靈活性高、可自訂處理邏輯 | 程式碼較複雜 |

#### 註解式使用範例
```java
@RestController
@RequestMapping("/coffee")
@RateLimiter(name = "coffee")  // 類別級別註解
public class CoffeeController {
    // 所有方法都會受到 RateLimiter 保護
}
```

#### 程式化使用範例
```java
@RestController
public class CoffeeOrderController {
    private RateLimiter rateLimiter;
    
    public CoffeeOrderController(RateLimiterRegistry rateLimiterRegistry) {
        rateLimiter = rateLimiterRegistry.rateLimiter("order");
    }
    
    @GetMapping("/{id}")
    public CoffeeOrder getOrder(@PathVariable("id") Long id) {
        try {
            // 使用 RateLimiter 包裝業務邏輯
            return rateLimiter.executeSupplier(() -> orderService.get(id));
        } catch(RequestNotPermitted e) {
            log.warn("請求被限流! {}", e.getMessage());
            return null;
        }
    }
}
```

### 監控與指標

#### 健康檢查端點
```bash
# 查看整體健康狀況
curl http://localhost:8080/actuator/health

# 查看 RateLimiter 健康狀況
curl http://localhost:8080/actuator/health/rateLimiter
```

#### 指標監控
```bash
# 查看所有指標
curl http://localhost:8080/actuator/metrics

# 查看 RateLimiter 相關指標
curl http://localhost:8080/actuator/metrics/resilience4j.ratelimiter.calls
```

### 環境變數配置
```properties
# 資料庫配置
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver

# JPA 配置
spring.jpa.hibernate.ddl-auto=none
spring.jpa.show-sql=true
spring.jpa.format-sql=true

# Actuator 配置
management.endpoints.web.exposure.include=*
management.endpoint.health.show-details=always
# 確保 Info 端點顯示所有 info.* 屬性，這是 Spring Boot 3.x 的行為，在 Spring Boot 2.x 中不需要這個設定
management.info.env.enabled=true

# 服務埠號
server.port=8080
```

## 參考資源

- [Resilience4j 官方文件](https://resilience4j.readme.io/)
- [Spring Boot Actuator 指南](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html)
- [Spring Cloud Consul 文件](https://spring.io/projects/spring-cloud-consul)
- [Joda Money 使用指南](https://www.joda.org/joda-money/)

## 注意事項與最佳實踐

### ⚠️ 重要提醒

| 項目 | 說明 | 建議做法 |
|------|------|----------|
| **版本相容性** | Resilience4j 2.2.0 配置語法變更 | 使用 `instances` 而非 `limiters` |
| **時間單位** | 配置檔案中的時間格式 | 使用毫秒數值而非 Duration 格式 |
| **例外處理** | RequestNotPermitted 例外處理 | 提供友善的錯誤回應給使用者 |
| **監控設定** | 生產環境監控配置 | 啟用事件訂閱和健康檢查指標 |

### 🔒 最佳實踐指南

- **合理設定限流參數**：根據系統容量和業務需求調整 `limit-for-period` 和 `limit-refresh-period`
- **提供降級機制**：當觸發限流時，提供快取資料或預設回應
- **監控告警設定**：設定適當的監控指標和告警閾值
- **測試驗證**：定期進行壓力測試，驗證限流配置的有效性
- **文件維護**：詳細記錄限流配置的業務邏輯和調整歷史

### 常見問題解決

#### Q: RateLimiter 沒有生效怎麼辦？
A: 檢查以下項目：
1. 確認使用 `instances` 而非 `limiters` 配置
2. 驗證 Resilience4j 自動配置是否正確載入
3. 檢查註解是否放在正確的位置

#### Q: 如何調整限流參數？
A: 
1. 監控當前系統負載和回應時間
2. 逐步調整 `limit-for-period` 參數
3. 觀察系統行為變化
4. 記錄最佳配置參數

## 授權說明

本專案採用 MIT 授權條款，詳見 LICENSE 檔案。

## 關於我們

我們主要專注在敏捷專案管理、物聯網（IoT）應用開發和領域驅動設計（DDD）。喜歡把先進技術和實務經驗結合，打造好用又靈活的軟體解決方案。

## 聯繫我們

- **FB 粉絲頁**：[風清雲談 | Facebook](https://www.facebook.com/profile.php?id=61576838896062)
- **LinkedIn**：[linkedin.com/in/chu-kuo-lung](https://www.linkedin.com/in/chu-kuo-lung)
- **YouTube 頻道**：[雲談風清 - YouTube](https://www.youtube.com/channel/UCXDqLTdCMiCJ1j8xGRfwEig)
- **風清雲談 部落格**：[風清雲談](https://blog.fengqing.tw/)
- **電子郵件**：[fengqing.tw@gmail.com](mailto:fengqing.tw@gmail.com)

---

**📅 最後更新：2025-08-27**  
**👨‍💻 維護者：風清雲談團隊**

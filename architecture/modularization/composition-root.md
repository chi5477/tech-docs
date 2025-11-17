# 相依注入與組合根（Composition Root）

> 組合根負責於應用程式啟動時完成相依關係的組裝（wiring）。相依注入應以建構子注入為主、介面導向、在最外層進行組裝，避免核心層出現框架耦合。

---

## 原則
- 介面優先：用例/服務依賴介面（Ports/Interfaces），實作位於 Adapters/Infrastructure。
- 建構子注入：將依賴透過建構子明確表達，避免 Service Locator 反模式。
- 組合根所在：`src/app/di/` 或等價的應用層啟動模組；測試有獨立組合根。
- 範疇（Scope）：依使用情境定義單例/請求範疇/工作範疇，避免過度單例化。
- 橫切關注：在組合根註冊攔截器/裝飾器（記錄、重試、快取、追蹤）。

---

## 基本流程
1) 宣告介面於核心層（Domain/Application）。
2) 在外層提供實作（Adapter/Infrastructure）。
3) 在組合根綁定介面→實作，並將綁定後的用例/控制器暴露給入站介面。

---

## 範例（語言無關概念）

介面（Port）：
```ts
// src/application/ports/PaymentProvider.ts
export interface PaymentProvider {
  charge(amount: number, currency: string, ref: string): Promise<string>;
}
```

用例：
```ts
// src/application/use-cases/ChargeOrder.ts
import { PaymentProvider } from "../ports/PaymentProvider";

export class ChargeOrder {
  constructor(private readonly payments: PaymentProvider) {}
  async exec(orderId: string) {
    // ... 檢索訂單、計算金額
    return this.payments.charge(100, "USD", orderId);
  }
}
```

Adapter 實作：
```ts
// src/adapters/outbound/StripePayment.ts
import { PaymentProvider } from "../../application/ports/PaymentProvider";

export class StripePayment implements PaymentProvider {
  constructor(private readonly apiKey: string) {}
  async charge(amount: number, currency: string, ref: string) {
    // 呼叫 Stripe SDK / HTTP
    return `tx_${ref}`;
  }
}
```

組合根：
```ts
// src/app/di/container.ts
import { ChargeOrder } from "../application/use-cases/ChargeOrder";
import { StripePayment } from "../adapters/outbound/StripePayment";

export function buildContainer() {
  const payments = new StripePayment(process.env.STRIPE_KEY!);
  const chargeOrder = new ChargeOrder(payments);
  return { chargeOrder };
}
```

入站介面（例如 HTTP 控制器）取得用例：
```ts
// src/interface/http/OrdersController.ts
export function OrdersController({ chargeOrder }: { chargeOrder: any }) {
  return {
    async postCharge(req, res) {
      const id = await chargeOrder.exec(req.params.id);
      res.json({ id });
    },
  };
}
```

---

## 測試與替身
- 單元測試：以假物件（Fake/Stub）或 Mock 實作 `PaymentProvider` 傳入用例建構子。
- 合約測試：對 `PaymentProvider` 介面與 `StripePayment` 的互動撰寫合約測試。
- 組合根覆寫：在測試組合根以測試用 Adapter 替換正式實作。

---

## 反模式警示
- 在核心層取得全域容器（Service Locator）。
- 控制器中直接 new 出具體實作，導致難以測試與替換。
- 將 ORM/HTTP Client 傳入領域模型、值物件或實體（污染核心層）。

---

## 檢核表
- [ ] 介面定義於核心層，實作位於外層。
- [ ] 建構子注入是主要注入方式。
- [ ] 組合根集中於 `src/app/di/` 並可於測試覆寫綁定。
- [ ] 橫切關注（追蹤/記錄/重試）以裝飾器或攔截器在組合根掛載。


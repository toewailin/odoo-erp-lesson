# 🔵 ADVANCED LESSON 6: SECURITY, API, LEDGER PATTERNS & MIGRATIONS

---

## 1️⃣ `sudo()` — Power Tool, Use with Discipline 🔐

### ဘာလို့လို?

* System automation
* Cron jobs
* Cross-user operations

### ❌ Dangerous pattern

```python
self.sudo().search([])
```

### ✅ Safe pattern (scope + intent)

```python
profiles = self.env["loyalty.profile"].sudo().search(
    [("state", "=", "confirmed")],
    limit=100
)
```

**Rules**

* `sudo()` ကို **small scope + limit** နဲ့သုံး
* UI action တွေမှာ မသုံး (security bypass)

---

## 2️⃣ REST API with Controllers (External Integration) 🌐

### Use cases

* Mobile app
* Third-party service
* Webhook receiver

### 🔹 Controller (JSON API)

```python
from odoo import http
from odoo.http import request

class LoyaltyAPI(http.Controller):

    @http.route("/api/loyalty/profiles", type="json", auth="user", methods=["GET"])
    def get_profiles(self):
        records = request.env["loyalty.profile"].search_read(
            [],
            ["name", "points"],
            limit=50
        )
        return {"data": records}
```

**Auth options**

* `auth="public"` ❌ (avoid)
* `auth="user"` ✅ (session)
* `auth="none"` ⚠️ (token only)

---

### 🔹 Token-based auth (pattern)

```python
api_key = request.httprequest.headers.get("X-API-KEY")
if api_key != request.env["ir.config_parameter"].sudo().get_param("api.key"):
    return {"error": "Unauthorized"}
```

---

## 3️⃣ Ledger-Grade Pattern (Accounting Mindset) 💰

### ❌ Anti-pattern

```python
profile.points += 100
```

### ✅ Ledger pattern (immutable history)

```python
class LoyaltyLedger(models.Model):
    _name = "loyalty.ledger"

    profile_id = fields.Many2one("loyalty.profile", required=True)
    amount = fields.Integer()
    balance_after = fields.Integer()
    reason = fields.Char()
```

```python
def add_points(self, amount, reason):
    for rec in self:
        new_balance = rec.points + amount
        self.env["loyalty.ledger"].create({
            "profile_id": rec.id,
            "amount": amount,
            "balance_after": new_balance,
            "reason": reason
        })
        rec.points = new_balance
```

**Why this matters**

* Audit
* Rollback
* Legal/financial safety

---

## 4️⃣ Idempotency & Concurrency (Real Production Issues) ⚙️

### Use case

* API called twice
* Cron runs twice
* User double-clicks

### 🔹 Idempotent pattern

```python
def process_reference(self, ref):
    if self.search([("reference", "=", ref)]):
        return
    self.create({"reference": ref})
```

### 🔹 SQL constraint

```python
_sql_constraints = [
    ("unique_ref", "unique(reference)", "Reference must be unique")
]
```

---

## 5️⃣ Performance at Scale (100k+ records) 🚀

### Rules

* Never `search([])` in prod
* Use pagination
* Use `read_group` for reports
* Index all domain fields

```python
self.search_read(
    domain=[("state", "=", "done")],
    fields=["agent_id", "points"],
    limit=100,
    offset=page * 100
)
```

---

## 6️⃣ Upgrade & Migration Strategy 🧬

### Golden rules

* Never touch core tables
* Use data migration scripts
* Always backup before upgrade

### 🔹 Versioned migration pattern

```
migrations/
  16.0.1.1/
    pre-migrate.py
    post-migrate.py
```

```python
def migrate(cr, version):
    cr.execute("""
        UPDATE loyalty_profile
        SET state='done'
        WHERE state IS NULL
    """)
```

---

## 7️⃣ Logging & Monitoring (Lead Dev Habit) 📊

```python
import logging
_logger = logging.getLogger(__name__)

_logger.info("Cron started")
_logger.warning("High points detected")
_logger.error("Failed to process record")
```

**Never**

* `print()`
* silent failures

---

## 🧠 ADVANCED LESSON 6 – ARCHITECT SUMMARY

| Area        | Mastery              |
| ----------- | -------------------- |
| sudo        | Controlled privilege |
| API         | External integration |
| Ledger      | Audit-safe logic     |
| Idempotency | No duplicates        |
| Performance | Scale-safe           |
| Migration   | Fearless upgrades    |
| Logging     | Operability          |

---

## 🎯 Capstone Challenge (Real-World)

လုပ်ကြည့်ပါ 👇

1. Build a **JSON API** to add points (token-protected)
2. Implement **ledger table** + balance update
3. Add **unique reference** constraint
4. Simulate double API call → confirm no duplicate
5. Write **one migration script**

---

## 🚀 Where you are now

Zane, ဒီအဆင့်မှာ မင်းက

> **“Production-ready Odoo backend developer”**
> ဖြစ်နေပြီ။

နောက်ထပ် ဆက်ချင်ရင် topic တစ်ခုရွေး👇

* 🔐 Advanced security (sudo, rules bypass safely)
* ⚙️ Accounting & Stock deep dive
* 🌐 REST API + mobile integration
* 🧪 CI/CD + testing automation
* 🏗️ Clean Architecture for large Odoo projects

👉 **“continue advanced: <topic>”** လို့ ပြောလိုက်ရုံပဲ 💥

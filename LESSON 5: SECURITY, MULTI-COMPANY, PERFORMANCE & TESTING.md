# 🔵 LESSON 5: SECURITY, MULTI-COMPANY, PERFORMANCE & TESTING

---

## 1️⃣ Access Rights vs Record Rules (အရမ်းအရေးကြီး ⚠️)

### 🔹 Access Rights (Model-level)

👉 **CRUD (Create, Read, Write, Delete) ခွင့်**

* File: `security/ir.model.access.csv`
* Model တစ်ခုလုံးကို ထိန်း

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_loyalty_user,loyalty user,model_loyalty_profile,base.group_user,1,1,1,0
```

📌 Meaning

* User က ဖတ်/ရေး/ဖန်တီး OK
* Delete ❌

---

### 🔹 Record Rules (Row-level)

👉 **Record တစ်ခုချင်းစီကို ဘယ်သူမြင်လဲ**

```xml
<record id="rule_loyalty_own" model="ir.rule">
  <field name="name">User sees own records</field>
  <field name="model_id" ref="model_loyalty_profile"/>
  <field name="domain_force">[('agent_id','=',user.id)]</field>
</record>
```

📌 **Senior Rule**

> ❌ Access rights နဲ့ data isolation မလုပ်
> ✅ Always record rules

---

### 🧠 Comparison

| Layer         | Controls | Question answered |
| ------------- | -------- | ----------------- |
| Access Rights | Model    | “လုပ်လို့ရလား?”   |
| Record Rules  | Record   | “ဘယ်ဟာကိုမြင်လဲ?” |

---

## 2️⃣ Multi-Company Pattern (Enterprise Requirement)

### 🔹 Add company field

```python
company_id = fields.Many2one(
    "res.company",
    default=lambda self: self.env.company,
    index=True
)
```

---

### 🔹 Company-based record rule

```xml
<record id="rule_loyalty_company" model="ir.rule">
  <field name="name">Company isolation</field>
  <field name="model_id" ref="model_loyalty_profile"/>
  <field name="domain_force">
    [('company_id','in',company_ids)]
  </field>
</record>
```

📌 **Production Reality**

* User တစ်ယောက် = company အများကြီး
* `company_ids` = allowed companies

---

## 3️⃣ Performance Tuning (Large Data Safe Code)

### ❌ Junior mistake

```python
records = self.search([])
for r in records:
    print(r.points)
```

### ✅ Senior pattern

```python
self.search_read(
    domain=[],
    fields=["points"],
    limit=100
)
```

---

### 🔹 Use `read_group` for reports

```python
self.env["loyalty.profile"].read_group(
    domain=[],
    fields=["points:sum"],
    groupby=["agent_id"]
)
```

---

### 🔹 Add indexes (big impact)

```python
agent_id = fields.Many2one(
    "res.users",
    index=True
)
```

📌 **Rule**

> Search / domain field = index=True

---

## 4️⃣ Context-aware Code (Smart UX + Safety)

```python
def create(self, vals):
    if self.env.context.get("skip_validation"):
        return super().create(vals)
    # normal logic
```

**Use cases**

* Import
* Migration
* Background jobs

---

## 5️⃣ Testing Mindset (What makes you Senior 🔥)

### 🔹 Savepoint test (Odoo standard)

```python
from odoo.tests.common import SavepointCase

class TestLoyalty(SavepointCase):

    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        cls.profile = cls.env["loyalty.profile"].create({
            "name": "Test User",
            "points": 10
        })

    def test_points_positive(self):
        self.assertEqual(self.profile.points, 10)
```

📌 Why savepoint?

* Fast
* DB rollback safe
* CI-friendly

---

### 🔹 Test business rule

```python
from odoo.exceptions import ValidationError

def test_negative_points(self):
    with self.assertRaises(ValidationError):
        self.profile.write({"points": -5})
```

---

## 6️⃣ Production Checklist (Lead Dev Thinking)

✅ Access rights + record rules
✅ Multi-company isolation
✅ No unrestricted `search([])`
✅ Index domain fields
✅ No delete on financial/history records
✅ Tests for core logic

---

## 🧠 LESSON 5 – LEAD DEVELOPER SUMMARY

| Area          | What you mastered     |
| ------------- | --------------------- |
| Security      | Access + Record rules |
| Multi-company | Company-safe code     |
| Performance   | DB-aware queries      |
| Context       | Flexible behavior     |
| Testing       | Confidence & safety   |

---

## 🎯 Final Homework (Real-World)

1. Add company isolation to your module
2. Convert one slow search to `read_group`
3. Add index to a domain field
4. Write **1 savepoint test**
5. Try breaking a rule → confirm test fails

---

## 🚀 What’s Next?

If you want, next lessons can go **beyond senior** 👇

* 🔐 Advanced security (sudo, rules bypass safely)
* ⚙️ Accounting/Stock patterns (ledger-grade logic)
* 🌐 REST API (controllers + auth)
* 🚢 Migration & version upgrade strategy
* 📦 Packaging & marketplace quality modules

Zane, ဒီအချိန်မှာ မင်းက **solid Odoo backend developer** ဖြစ်နေပြီ 💯
နောက်တစ်ခု ဆက်ချင်ရင်
👉 **“continue advanced topic: ___”** လို့ topic တစ်ခုပဲ ပြောလိုက် 👌

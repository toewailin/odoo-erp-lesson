# 🔵 LESSON 2: BUSINESS LOGIC & DATA CONTROL (CORE SKILLS)

ဒီ lesson မှာ အောက်ပါ 4 ခုကို သေချာပိုင်ဆိုင်အောင်လုပ်မယ် 👇

1. `create()` / `write()` override
2. `@api.onchange`
3. Record Rules (multi-agent isolation)
4. Scheduled Actions (Cron jobs)

---

## 1️⃣ `create()` & `write()` Override

👉 **Professional Odoo dev မဖြစ်မနေသုံးရတဲ့ skill**

### ဘာကြောင့် override လုပ်ရလဲ?

* auto reference number ထုတ်ချင်
* audit log သိမ်းချင်
* business rule enforce လုပ်ချင်
* ledger / history record ဖန်တီးချင်

---

### 🔹 Example: Auto Code + Log on Create

```python
from odoo import api, fields, models

class LoyaltyProfile(models.Model):
    _name = "loyalty.profile"

    name = fields.Char(required=True)
    code = fields.Char(readonly=True)
    points = fields.Integer(default=0)

    @api.model
    def create(self, vals):
        if not vals.get("code"):
            vals["code"] = self.env["ir.sequence"].next_by_code("loyalty.profile")
        record = super().create(vals)

        # audit log
        self.env["mail.message"].create({
            "model": self._name,
            "res_id": record.id,
            "body": f"Profile created with {record.points} points"
        })
        return record
```

📌 **Professional Notes**

* `super().create(vals)` ကို မမေ့ ❗
* `self.env` = Odoo environment (DB, user, context)
* Sequence = ERP standard pattern

---

### 🔹 Override `write()` (Update control)

```python
def write(self, vals):
    for rec in self:
        old_points = rec.points
        res = super().write(vals)

        if "points" in vals:
            rec.message_post(
                body=f"Points changed: {old_points} → {rec.points}"
            )
    return res
```

📌 **Use case**

* salary change log
* stock adjustment log
* wallet balance history

---

## 2️⃣ `@api.onchange`

👉 **Form UI behavior (frontend logic)**

### ဘာအတွက်သုံးလဲ?

* user data ထည့်နေချိန် validation
* auto fill fields
* warning message ပြ

---

### 🔹 Example: Warning when points too high

```python
@api.onchange("points")
def _onchange_points(self):
    if self.points and self.points > 1000:
        return {
            "warning": {
                "title": "High Points",
                "message": "Points exceed normal limit!"
            }
        }
```

📌 **Important**

* `onchange` = DB မသိမ်းဘူး
* UI behavior only

---

## 3️⃣ Record Rules (Multi-Agent Data Isolation) 🔐

👉 **ERP security ရဲ့ အသက်**

### Scenario (အရမ်းအရေးကြီး)

* Agent A ❌ Agent B ရဲ့ player မမြင်ရ
* User တစ်ယောက်က ကိုယ့် data ပဲမြင်ရ

---

### 🔹 Add agent field

```python
agent_id = fields.Many2one(
    "res.users",
    string="Agent",
    default=lambda self: self.env.user
)
```

---

### 🔹 Record Rule XML

```xml
<record id="rule_loyalty_profile_agent" model="ir.rule">
  <field name="name">Agent can see own profiles</field>
  <field name="model_id" ref="model_loyalty_profile"/>
  <field name="domain_force">[('agent_id','=',user.id)]</field>
  <field name="groups" eval="[(4, ref('base.group_user'))]"/>
</record>
```

📌 **Professional Rule**

> Security ကို Python မှာမဟုတ်
> **Always Record Rules + Access CSV** နဲ့ထိန်း

---

## 4️⃣ Scheduled Actions (Cron Jobs) ⏱️

👉 **Auto jobs / background logic**

### Use cases

* monthly reset
* auto expiry
* report generation
* interest calculation

---

### 🔹 Python Method

```python
def cron_reset_points(self):
    profiles = self.search([])
    for p in profiles:
        p.points = 0
```

---

### 🔹 XML Cron

```xml
<record id="cron_reset_loyalty_points" model="ir.cron">
  <field name="name">Reset Loyalty Points</field>
  <field name="model_id" ref="model_loyalty_profile"/>
  <field name="state">code</field>
  <field name="code">model.cron_reset_points()</field>
  <field name="interval_number">1</field>
  <field name="interval_type">months</field>
  <field name="numbercall">-1</field>
</record>
```

📌 **Production Tip**

* Heavy job → batch process
* Always test cron in staging first

---

# 🧠 LESSON 2 – PROFESSIONAL SUMMARY

| Skill        | Why it matters             |
| ------------ | -------------------------- |
| create/write | Business logic enforcement |
| onchange     | UX quality                 |
| record rules | Security & data isolation  |
| cron         | Automation                 |

---

## 🎯 Homework (Real Developer Task)

လုပ်ကြည့်ပါ 👇

1. `points` negative ဖြစ်ရင် create/write မှာ error throw
2. `points > 500` ဖြစ်ရင် onchange warning ပြ
3. Agent record rule သုံးပြီး login test
4. Cron job ကို manual run ပြီး log confirm

---

Zane 🔥
နောက် Lesson 3 မှာ
👉 **Many2one / One2many / Many2many deep dive**
👉 **Smart buttons + chatter**
👉 **Performance tips (search vs read_group)**

ကို **senior Odoo developer mindset** နဲ့ ဆက်သင်မယ်။

ဆက်ချင်ရင် **“continue lesson 3”** လို့ ပြောလိုက်ရုံပဲ 👌

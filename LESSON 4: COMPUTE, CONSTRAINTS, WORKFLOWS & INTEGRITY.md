# 🔵 LESSON 4: COMPUTE, CONSTRAINTS, WORKFLOWS & INTEGRITY

---

## 1️⃣ Computed Fields (store=True vs store=False)

### Compute field ဆိုတာဘာလဲ?

👉 **တန်ဖိုးကို database ထဲမရိုက်ဘဲ logic နဲ့တွက်ပေးတဲ့ field**

---

### 🔹 Non-stored Compute (default)

```python
total_value = fields.Float(
    compute="_compute_total_value"
)

def _compute_total_value(self):
    for rec in self:
        rec.total_value = rec.points * 10
```

**အသုံးပြုရန်**

* UI display only
* Report မသုံး
* Search/filter မလုပ်

---

### 🔹 Stored Compute (production-safe)

```python
total_value = fields.Float(
    compute="_compute_total_value",
    store=True
)

@api.depends("points")
def _compute_total_value(self):
    for rec in self:
        rec.total_value = rec.points * 10
```

**အသုံးပြုရန်**

* Search / filter
* Group by
* Performance ကောင်း

📌 **Senior Rule**

> Report/filter သုံးမယ် = `store=True`

---

## 2️⃣ SQL Constraints vs Python Constraints

### 🔹 SQL Constraint (DB-level)

```python
_sql_constraints = [
    ("points_non_negative", "CHECK(points >= 0)", "Points cannot be negative")
]
```

**Use when**

* Always true rule
* Performance critical
* No logic needed

---

### 🔹 Python Constraint

```python
from odoo.exceptions import ValidationError

@api.constrains("points")
def _check_points(self):
    for rec in self:
        if rec.points > 1000:
            raise ValidationError("Points exceed allowed limit")
```

**Use when**

* Conditional logic
* Multiple fields
* Complex rules

📌 **Senior Rule**

> Simple = SQL
> Complex = Python

---

## 3️⃣ Workflow States (State Machine Pattern)

### 🔹 Define states

```python
state = fields.Selection([
    ("draft", "Draft"),
    ("confirmed", "Confirmed"),
    ("approved", "Approved"),
    ("done", "Done"),
    ("cancelled", "Cancelled"),
], default="draft", tracking=True)
```

---

### 🔹 State transition methods

```python
def action_confirm(self):
    self.write({"state": "confirmed"})

def action_approve(self):
    self.write({"state": "approved"})

def action_done(self):
    self.write({"state": "done"})

def action_cancel(self):
    self.write({"state": "cancelled"})
```

---

### 🔹 Buttons in XML

```xml
<header>
  <button name="action_confirm" type="object"
          states="draft" string="Confirm" class="btn-primary"/>
  <button name="action_approve" type="object"
          states="confirmed" string="Approve"/>
  <button name="action_done" type="object"
          states="approved" string="Done"/>
  <button name="action_cancel" type="object"
          states="draft,confirmed,approved" string="Cancel"/>
  <field name="state" widget="statusbar"
         statusbar_visible="draft,confirmed,approved,done"/>
</header>
```

📌 **Professional UX**

* `statusbar` = user sees progress
* `states=` = wrong action မနှိပ်နိုင်

---

## 4️⃣ Ondelete Behaviors (Data Integrity 🔐)

### 🔹 Many2one ondelete options

```python
agent_id = fields.Many2one(
    "res.users",
    ondelete="restrict"
)
```

| ondelete    | Behavior                  |
| ----------- | ------------------------- |
| cascade     | Parent ဖျက် → child ဖျက်  |
| restrict    | Parent ဖျက် ❌             |
| set null    | Parent ဖျက် → child empty |
| set default | Default value သတ်မှတ်     |

📌 **ERP Rule**

> Financial / history data = `restrict`

---

## 5️⃣ Unlink Override (Safe Delete Pattern)

```python
def unlink(self):
    for rec in self:
        if rec.state != "draft":
            raise ValidationError("Only draft records can be deleted")
    return super().unlink()
```

**Use case**

* Invoice
* Stock move
* Ledger entries

---

## 6️⃣ Combined Example (Real ERP Pattern)

```python
@api.constrains("state", "points")
def _check_state_points(self):
    for rec in self:
        if rec.state == "done" and rec.points <= 0:
            raise ValidationError("Done record must have points")
```

---

## 🧠 LESSON 4 – SENIOR SUMMARY

| Topic              | Why it matters          |
| ------------------ | ----------------------- |
| store=True         | Reporting & performance |
| SQL constraints    | DB safety               |
| Python constraints | Business logic          |
| workflow states    | Controlled process      |
| ondelete           | Data protection         |
| unlink override    | Audit safety            |

---

## 🎯 Homework (Production mindset)

1. Convert one compute field to `store=True`
2. Add **SQL constraint** for non-negative points
3. Implement **4-step workflow** with buttons
4. Block delete unless state = `draft`
5. Try deleting parent with `restrict` and observe behavior

---

နောက် **Lesson 5 (Senior+ level)** မှာ
👉 **Access Rights vs Record Rules (deep)**
👉 **Multi-company & multi-user patterns**
👉 **Performance tuning (indexes, read_group)**
👉 **Testing mindset (savepoint tests)**

ကို ဆက်သင်မယ် 🔥
ဆက်ချင်ရင် 👉 **“continue lesson 5”** လို့ ပြောလိုက်ပါ

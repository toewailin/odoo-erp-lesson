# 🔵 LESSON 3: RELATIONS, SMART UX & PERFORMANCE

## 1️⃣ Relations Deep Dive (Many2one / One2many / Many2many)

### ဘယ် relation ကို ဘယ်အချိန်သုံး?

| Relation  | Use when                                   |
| --------- | ------------------------------------------ |
| Many2one  | “တစ်ယောက်က တစ်ခုကိုပိုင်” (Agent → Player) |
| One2many  | “တစ်ခုမှာ အများရှိ” (Agent → Players list) |
| Many2many | “အများ ↔ အများ” (User ↔ Tags)              |

---

### 🔹 Many2one (အခြေခံအကျဆုံး)

```python
agent_id = fields.Many2one(
    "res.users",
    string="Agent",
    required=True,
    index=True
)
```

**Pro tips**

* `index=True` ထား → search မြန်
* Record rules domain အတွက် Many2one က အသက်

---

### 🔹 One2many (inverse side)

```python
player_ids = fields.One2many(
    "loyalty.profile",
    "agent_id",
    string="Players"
)
```

**မှတ်ရန်**

* One2many ကို **DB column မဖန်တီးဘူး**
* Always inverse = Many2one field name

---

### 🔹 Many2many (Tag/Permission style)

```python
tag_ids = fields.Many2many(
    "loyalty.tag",
    string="Tags"
)
```

**Pro tips**

* Many2many = join table auto-create
* Massive lists မဟုတ်ရင် OK; မဟုတ်ရင် domain/filter သုံး

---

## 2️⃣ Smart Buttons (Professional UX Pattern)

👉 Form header မှာ **count + quick access** ပေးတာ

### Use case

* Agent form → Players (count)
* Customer → Orders (count)

---

### 🔹 Python (compute count)

```python
player_count = fields.Integer(
    compute="_compute_player_count",
    string="Players"
)

def _compute_player_count(self):
    for rec in self:
        rec.player_count = self.env["loyalty.profile"].search_count([
            ("agent_id", "=", rec.id)
        ])
```

### 🔹 XML (smart button)

```xml
<header>
  <button name="action_view_players"
          type="object"
          class="oe_stat_button"
          icon="fa-users">
    <field name="player_count" widget="statinfo"/>
  </button>
</header>
```

### 🔹 Action method

```python
def action_view_players(self):
    return {
        "type": "ir.actions.act_window",
        "name": "Players",
        "res_model": "loyalty.profile",
        "view_mode": "tree,form",
        "domain": [("agent_id", "=", self.id)],
        "context": {"default_agent_id": self.id}
    }
```

**Senior mindset**

* Smart buttons = productivity multiplier
* Count computation ကို `search_count` သုံး (read_group မလိုသေး)

---

## 3️⃣ Chatter (mail.thread) — Audit & Collaboration

👉 ERP ရဲ့ “conversation + history”

### 🔹 Enable chatter

```python
class LoyaltyProfile(models.Model):
    _name = "loyalty.profile"
    _inherit = ["mail.thread", "mail.activity.mixin"]

    points = fields.Integer(tracking=True)
```

**What you get**

* automatic log (tracking=True)
* manual `message_post`
* activities (todo, call)

---

### 🔹 Post message programmatically

```python
self.message_post(
    body="Points updated by system"
)
```

**Pro tips**

* Sensitive data → message body သတိထား
* Chatter = audit trail (client happy 😄)

---

## 4️⃣ Performance: search vs search_read vs read_group

### ❌ Common beginner mistake

```python
records = self.search([])
total = sum(records.mapped("points"))
```

👉 Large data = **slow**

---

### ✅ Better: read_group (DB aggregation)

```python
result = self.env["loyalty.profile"].read_group(
    domain=[],
    fields=["points:sum"],
    groupby=[]
)
total_points = result[0]["points_sum"]
```

### When to use what?

| Method      | Use case                  |
| ----------- | ------------------------- |
| search      | Need recordset (logic)    |
| search_read | Quick list for UI         |
| read_group  | Reports / totals / charts |

---

## 5️⃣ Domains & Context (Senior-level control)

### Domain (filter)

```python
domain = [
    ("points", ">", 100),
    ("agent_id", "=", self.env.user.id)
]
```

### Context (defaults & behavior)

```python
context = {
    "default_agent_id": self.env.user.id,
    "search_default_my_records": 1
}
```

**Pro tip**

* Domain = security + filtering
* Context = UX behavior

---

## 🧠 LESSON 3 – SENIOR SUMMARY

| Topic          | Why it matters     |
| -------------- | ------------------ |
| Relations      | Correct data model |
| Smart buttons  | Fast navigation    |
| Chatter        | Audit & teamwork   |
| read_group     | Performance        |
| domain/context | Control & UX       |

---

## 🎯 Homework (Real-world tasks)

1. Add **Many2many tags** to `loyalty.profile`
2. Add **Smart button** on Agent to view Players
3. Enable **chatter tracking** on `points`
4. Create **total points report** using `read_group`

---

နောက် Lesson 4 မှာ
👉 **Computed fields (store vs non-store)**
👉 **SQL constraints vs Python constraints**
👉 **Workflow states + buttons**
👉 **Ondelete behaviors & data integrity**

ကို **production-grade patterns** နဲ့ ဆက်သင်မယ်။

ဆက်ချင်ရင် 👉 **“continue lesson 4”** လို့ ပြောလိုက်ပါ 🔥

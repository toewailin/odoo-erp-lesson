## 0) Odoo Developer Roadmap (သင်ယူမယ့်အချက်ကြီး 6 ခု)

1. **Odoo Architecture** (addons, models, ORM, views, actions)
2. **Python for Odoo** (classes, methods, decorators, recordsets)
3. **Models & Fields** (Many2one/One2many/Many2many, constraints)
4. **Views** (form/tree/search, menus, security)
5. **Business Logic** (compute, onchange, create/write overrides, workflows)
6. **Quality/Prod** (access rules, tests, performance)

---

## 1) Setup (လက်တွေ့ Project Start) — Standard Dev Environment

✅ Best practice stack:

* **Docker** (Odoo + Postgres)
* VS Code + Python extension
* git repo

### Docker Compose (မင်းရဲ့ local dev အတွက် pattern)

* odoo container
* postgres container
* volume for addons

> မင်း Docker/K8s သိပြီးသားဆိုတော့ ဒီ setup က “fast iteration” အတွက်အကောင်းဆုံး။

---

## 2) Odoo Module Structure (မဖြစ်မနေမှတ်)

Module တစ်ခုမှာ အနည်းဆုံး—

```
my_module/
  __init__.py
  __manifest__.py
  models/
    __init__.py
    my_model.py
  views/
    my_model_views.xml
  security/
    ir.model.access.csv
```

### **manifest**.py အဓိက

* name, depends, data (xml, access) ထည့်တယ်

---

## 3) First Real Module Example (အလုပ်ဖြစ်တဲ့ level)

**Goal:** “Player/Agent” လို data model တစ်ခုလုပ်ကြမယ် (betting မဟုတ်ပဲ ERP style example အနေနဲ့)
ဥပမာ: **Customer Loyalty Profile** (player-like) + **Partner Agent** (agent-like)

### models/loyalty_profile.py

```python
from odoo import api, fields, models
from odoo.exceptions import ValidationError

class LoyaltyProfile(models.Model):
    _name = "loyalty.profile"
    _description = "Loyalty Profile"
    _rec_name = "display_name"

    name = fields.Char(string="Name", required=True)
    phone = fields.Char(string="Phone")
    agent_id = fields.Many2one("res.partner", string="Agent", domain=[("is_company", "=", True)])
    points = fields.Integer(string="Points", default=0)

    display_name = fields.Char(string="Display Name", compute="_compute_display_name", store=True)

    @api.depends("name", "phone")
    def _compute_display_name(self):
        for rec in self:
            rec.display_name = f"{rec.name} ({rec.phone or 'no phone'})"

    @api.constrains("points")
    def _check_points(self):
        for rec in self:
            if rec.points < 0:
                raise ValidationError("Points cannot be negative.")
```

**Professional notes**

* `_name` = model technical name
* `fields.Many2one` က relation
* `@api.depends` = compute field
* `@api.constrains` = validation rule ✅

---

## 4) Views (UI ထုတ်တာ)

### views/loyalty_profile_views.xml

```xml
<odoo>
  <record id="view_loyalty_profile_tree" model="ir.ui.view">
    <field name="name">loyalty.profile.tree</field>
    <field name="model">loyalty.profile</field>
    <field name="arch" type="xml">
      <tree>
        <field name="display_name"/>
        <field name="agent_id"/>
        <field name="points"/>
      </tree>
    </field>
  </record>

  <record id="view_loyalty_profile_form" model="ir.ui.view">
    <field name="name">loyalty.profile.form</field>
    <field name="model">loyalty.profile</field>
    <field name="arch" type="xml">
      <form>
        <sheet>
          <group>
            <field name="name"/>
            <field name="phone"/>
            <field name="agent_id"/>
            <field name="points"/>
          </group>
        </sheet>
      </form>
    </field>
  </record>
</odoo>
```

---

## 5) Menus + Actions (Menu ထဲမှာ ပေါ်အောင်)

```xml
<odoo>
  <record id="action_loyalty_profile" model="ir.actions.act_window">
    <field name="name">Loyalty Profiles</field>
    <field name="res_model">loyalty.profile</field>
    <field name="view_mode">tree,form</field>
  </record>

  <menuitem id="menu_loyalty_root" name="Loyalty"/>
  <menuitem id="menu_loyalty_profiles" name="Profiles"
            parent="menu_loyalty_root"
            action="action_loyalty_profile"/>
</odoo>
```

---

## 6) Security (Beginner တွေလက်လွတ်တတ်တာ ⚠️)

### security/ir.model.access.csv

```
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_loyalty_profile_user,loyalty.profile user,model_loyalty_profile,base.group_user,1,1,1,0
```

* unlink = 0 ထားတာက “data မဖျက်နိုင်” လုပ်ချင်လို့

---

## 7) Next Lesson (အခုမှစပြီး Professional Level တင်မယ်)

နောက်တစ်ခါကနေ ဒီ 4 ခုကို ဆက်သင်မယ်:

1. **create() / write() override** (ledger-style logging)
2. **onchange** (form မှာ live behavior)
3. **record rules (multi-agent data isolation)**
4. **server actions + scheduled actions** (auto jobs)

---

## မင်းအတွက် “သင်တန်းပုံစံ” လိုလားတဲ့အတွက်

ဒီနေ့ Lesson 1 အနေနဲ့ မေးခွန်း ၃ ခုကို ဖြေပေး (အဲဒါနဲ့ မင်း project ဆီကို တိတိကျကျကူးပြောင်းပြီး သင်ပေးမယ်)

1. မင်း Odoo version ဘာသုံးနေလဲ? (16/17/18?)
2. မင်း local setup က Docker လား? Ubuntu/Windows လား?
3. မင်းလုပ်ချင်တဲ့ ERP module က “Asset Management” ပဲလား၊ အခြား module လည်းပါသေးလား?

မင်းပြန်ဖြေလိုက်တာနဲ့ **မင်းရဲ့ Modules/Core style နဲ့ကိုက်တဲ့ Odoo Clean structure** အတိုင်း lesson-by-lesson သင်ပေးမယ် ✅

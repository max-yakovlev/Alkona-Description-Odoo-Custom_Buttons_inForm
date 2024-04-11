# Alkona-Description-Odoo-Custom_Buttons_inForm
Цель:
Создать модальное окно, которое вызывается на форме по custom кнопке. В этом окне определить custom кнопку для вызова какого-либо метода с возможностью передать параметры в другую форму/модель для этого метода.   


**1.**
- И так у нас есть модель и форма. Не забываем зарегестривать View в ```__manifest__.py```

**Наша модель**    
```python
from odoo import models, fields, api


class TestModel(models.Model):
    _name = 'test_model'  # Имя модели по которой мы можем обращаться к ней 

    c_name = fields.Char(string='Наименование', size=800, required=True, index='trigram')
    c_code = fields.Char(string='Код', size=50)
                                 
    def test_action_form(self): # Метод возращает действие, которое вызвает новую форму. У этой формы должна быть своя модель, как вариант в нашем случаее временная - models.TransientModel, ее мы определим позже
        return {
                  'name' : 'Окошко с сообщением',
                  'type': 'ir.actions.act_window',
                  'view_mode': 'form',             
                  'view_type': 'form',            
                  'res_model': 'test.model.wizard',
                  'target': 'new',   # new - говорит и том, что нужно вызвать новое окно "модальное". Если опустить данный параметр откроется полноценная форма привязанная к модели 
               }
```
Кнопка в форме определяется просто:    

```xml
<odoo>
   <record id="test_model_form" model="ir.ui.view" >
         <field name="name">test.model.form</field>
         <field name="model">test_model</field> <!-- Модель для которой определенна форма, в ней будет содержаться метод на который мы будем ссылаться в кнопке -->
         <field name="type">form</field>
         <field name="arch" type="xml">
            <form>                              
                <field name="c_code" />
                <field name="c_name" />
                <button name="test_action_form" type="object" string="Notification" class="oe_highlight"/> <!-- Определяем кнопку. name - имя метода в модели -->                  
            </form>
         </field>
      </record>
</odoo>
```
- Чтобы вызвать эту форму нам нужно определить Action и Refference/Button на этот Action
-     
  **Создаем Action**   
  ```xml
  <odoo>
    <record id="test_model_action" model="ir.actions.act_window">
         <field name="name">Action Name</field>
         <field name="res_model">test_model</field>
         <field name="view_mode">form</field>         
    </record>
  </odoo>
  ```
     
  **Создаем Menu/Refference/Button**
  ```xml
  <odoo>
     <menuitem name="Refference/Button name" id="test_menu_root" groups="tfoms-base.group_tfoms_user" />        
        <menuitem name="Refference/Button name" id="test_sub_menu" action="test_model_action" parent="test_menu_root"/>
  </odoo>
  ```
  
  
**2.**
- Создаем временную модель и представление с кнопками для нее
- 
  **Временная модель**
  
  ```python
  from odoo import models, fields, api
  
    class TestMessag(models.TransientModel): # Указываем, что модель временная
    _name = 'test-model-wizard'

    message = fields.Char(string='Сообщение', readonly='true', default='Любая строка по умолчанию')
    param1 = fields.Selection([('param1','Param1'), # Выпадающий список, где первый параметр = значение, второй = отображаемое имя параметра в UI
                               ('param2','Param2'),
                               ('param3','Param3')])
    # Метод который будет вызываться из модального окна/формы и передавать параметры для целевого окна/формы
    # Предположим, что мы создали целевую модель с формой и ей нужно передать некие параметры                                    
    def do_job(self):
        # Создаем словарь со значениями
        vals = {
            'message' = self.message,
            'param1' = self.param1
        }
          # С помощью odoo ORM метода create() создаем запись(экземпляр) уже существующей модели.
          # Передаем в ее поля значения, будем считать что они соотвествуют 'message', 'param1'
          # И помещаем созданную запись в переменную target_model_rec. Id этой записи создается системой
          # Далее мы можем обратиться к переменной этой записи и получить ее Id, что и делаем в return вызвывая форму/модель с этой записью
        target_model_rec = self.env['имя целевой модели'].create(vals)
        return{
              'name': 'Name',                  # Произвольное имя
              'type': 'ir.actions.act_window',
              'view_mode': form,               # Говорим, что это форма
              'res_model': 'target_model',     # модель целевой формы
              'res_id': 'target_model.rec.id', # сообщаем, что хотим получить форму с данными по этому Id 
            }
  ```
  **Преставление/View**
  
  ```xml
      <?xml version="1.0" encoding="UTF-8"?>
  <odoo>
   <data>
      <record model="ir.ui.view" id="get_modal_test">
         <field name="name">get.modal.test</field>
         <field name="model">test-model-wizard</field>
         <field name="type">form</field>
         <field name="arch" type="xml">
            <form>               
               <group colspan="4" col="4">
                  <field name="message" />
                  <field name="param1" /> 
               </group>
               <footer>
                  <button string="Текст в кнопке" class="oe_highlight" special="cancel"/>  <!-- special="cancel" - говорит о том, что при нажатии модальное окно будет закрыто  -->
                  <button name="do_job" string="Текст в кнопке" type="object" class="oe_highlight"/> <!-- name="do_job" - ссылаемся на метод во временной модели  -->
               </footer>
            </form>
         </field>
      </record>      
   </data>
  </odoo>
  ```

# Developer Cheatsheet

### Server Side

#### Frappe Module

##### `frappe.form_dict`
- Request parameters.

##### `frappe.get_doc(doctype, name)`
- Example: `frappe.get_doc('Project', 'My Project')`
- Loads a document from the database with the given doctype (table) and name.
- Returns a Document object where all columns are properties, e.g., `doc.name`.

##### `frappe.get_meta(doctype)`
- Loads the metadata (DocType) of the given doctype.
- Example: `frappe.get_meta('Task').fields` is the list of fields in the Task doctype.

##### `frappe.get_all(doctype, filters, fields)`
- Example: `frappe.get_all('Project', filters={'status': 'Open'}, fields=['name', 'description'])`
- Returns a list of dict objects from the database.

##### `frappe.get_list(doctype, filters, fields, order_by)`
- Similar to `frappe.get_all` but only shows records permitted for the user.
- Allows sorting with the `order_by` parameter.
- Example:
  ```python
  frappe.get_list('Payment Entry', 
                  filters={'docstatus': 0, 'payment_type': 'Pay'}, 
                  fields=['name', 'posting_date', 'paid_amount'], 
                  order_by='posting_date')
  ```
- Use `limit_page_length` to increase results and `limit_start` for pagination.

##### `frappe.get_value(doctype, name, fieldname)`
- Example: `frappe.get_value('Task', 'TASK00030', 'owner')`
- Returns a single value from the database.

##### `frappe.get_last_doc(doctype)`
- Example: `frappe.get_last_doc('Project')`
- Gets the last created document of this type.

##### `frappe.get_single(doctype)`
- Example: `frappe.get_single('Dropbox Settings')`
- Returns a `frappe.model.document.Document` object of the given Single Doctype.

##### `frappe.get_installed_apps()`
- Returns a list of all installed apps in the current site.

#### Document Object

##### Load a Document
```python
doc = frappe.get_doc(doctype, name)

# Get properties
doc.title

# Set properties
doc.first_name = 'My Name'

# Save the document
doc.save()
```

##### Insert a New Document
```python
doc = frappe.get_doc({
    "doctype": "Project",
    "title": "My new project",
    "status": "Open"
})
doc.insert()
```

#### How to Make Public API

Add `@frappe.whitelist()` to the function. Example:

```python
@frappe.whitelist()
def get_last_project():
    return frappe.get_all("Project", limit_page_length=1)[0]
```

Accessible as `/api/method/myapp.api.get_last_project`. Example JS call:

```javascript
frappe.call({
    method: "myapp.api.get_last_project",
    callback: (response) => {
        console.log(response.message);
    }
});
```

#### Debugging Syntax Errors
Run the following commands to debug:
```bash
cd ~/frappe-bench/sites
../env/bin/python ../apps/frappe/frappe/utils/bench_helper.py
```

---

### Client Side

#### Desk Globals
- **`cur_frm`**: Current form object.
- **`cur_list`**: Current list object.
- **`cur_dialog`**: Current open dialog.
- **`cur_page`**: Current page object.
- **`frappe.quick_entry`**: Current Quick Entry object.
- **`locals`**: All documents and DocTypes loaded in the browser session. Example:
  ```javascript
  locals['Opportunity']['OTY00001']
  ```

#### Routing
- Standard routes:
  - `#List/[doctype]`
  - `#Form/[doctype]/[name]`
  - `#Report/[doctype]`
  - `#Calendar/[doctype]`
  - `#Tree/[doctype]`
  - `#modules/[module_name]`
  - `#activity`
  - `#Dashboard`

- Change route via JS:
  ```javascript
  frappe.set_route("List", "Customer");
  ```

- Pass values to a view using `frappe.route_options`. Example:
  ```javascript
  frappe.route_options = {"customer_type": "Company"};
  frappe.set_route("List", "Customer");
  ```

---

### Forms

#### Form API

1. Add a handler on value change:
```javascript
frappe.ui.form.on("Sales Order", {
    company: function(frm) {
        // Triggered when the company field is modified.
    },
    onload: function(frm) {
        // Triggered when the document is loaded.
    }
});
```

2. Adding Standard JS Listeners:
```javascript
frappe.ui.form.on("Sales Invoice", {
    onload_post_render: function(frm) {
        cur_frm.fields_dict.customer.$input.on("keypress", function(evt) {
            // Code here runs on keypress in the customer field.
        });
    }
});
```

3. Change form value:
```javascript
frm.set_value(fieldname, value);
```

4. Database interactions:
```javascript
frappe.db.get_value(doctype, filters, fieldname, callback);
frappe.db.insert({
    doctype: 'Project',
    project_name: 'Build a decent documentation'
});
```

5. Add a new row to a child table:
```javascript
frappe.ui.form.on("Project", {
    onload: function(frm) {
        if (frm.is_new()) {
            frm.add_child('tasks').title = 'Update Developer Cheatsheet';
            frm.refresh_field('tasks');
        }
    }
});
```

---

### Running Tasks Serially
```javascript
frappe.run_serially([
  () => frappe.set_route('List', 'ToDo'),
  () => frappe.new_doc('ToDo')
]);
```

---

### Tests
Run individual tests:
```bash
bench --site test-service run-tests --module erpnext.tests.test_woocommerce
```

---

### Utility
To get a list of installed apps:
```bash
bench --site [sitename] console
frappe.get_installed_apps()

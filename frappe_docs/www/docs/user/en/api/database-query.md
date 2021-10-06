---
add_breadcrumbs: 1
title: DatabaseQuery - API
metatags:
 description: >
  API methods for querying records in Frappe
---

# DatabaseQuery

It's the engine powers the List, Report and some of the favourite developer APIs like `frappe.get_all` and `frappe.get_list`

## Filters

DatabaseQuery likes it's filters in a specific format....okay, it supports various different formats, but it generally looks something like this =>

> Table > Column > Operator > Value

So here Table may be optional. So you'd have a filter (one where condition) look like [["column", "=", "value"]] or {"column": "value"}. Because of this restriction, you can only ever compare columns to values. This PR aims to add support for Column to Column comparison.

There's a few broader ways of writing filters

### Dict Notation

Find out which documents have been modified after their initial creation (Compare two datetime columns)
Check if search_field value has been used in description (Check if dynamic value in document is used elsewhere [like])
In [1]: from frappe.query_builder import Column

In [2]: frappe.get_all("Note", {"modified": (">", Column("creation"))})
Out[2]: [{'name': 'SUjXJ1Wa0R'}, {'name': 'tUSNajSteH'}, {'name': '2sC3n9l0N0'}]

> TODO: Add introductory content & examples

### List Notation

> TODO: Add introductory content & examples

### Filter Object Notation


This allows ```frappe.qb``` objects to be passed directly as filters.

In [1] test_table = frappe.qb.DocType("Test Table")

In [2]: frappe.db.delete(test_table, filters=(test_table.name=="TestUser") | (test_table.age==10), run=False)

Out [2]: DELETE FROM "tabTest Table" WHERE "name"=\'TestUser\' OR "age"=10

In [3]: test_table = frappe.qb.DocType("User")

In [4]: frappe.db.get_value(test_table,
		filters=(test_table.email=="admin@localhost.com") | (test_table.name.like("Administrator")),
		fieldname=["name"], debug=True)
out [4]: SELECT "name" FROM "tabUser" WHERE "email"='example@localhost.com' OR "name" LIKE 'Example'
		 Execution time: 0.1 sec
		 'Administrator'

## frappe.db.get_list

`frappe.db.get_list(doctype, filters, or_filters, fields, order_by, group_by, start, page_length)`

- Also aliased to `frappe.get_list`

Returns a list of records from a `doctype` table. ORM Wrapper for a `SELECT`
query. Will also apply user permissions for the records for the session user. Only returns the document names if the `fields` keyword argument is not given. By default this method returns a list of `dict`s, but, you can pluck a particular field by giving the `pluck` keyword argument:

```python
frappe.db.get_list('Employee')

# output
[{'name': 'HR-EMP-00008'},
 {'name': 'HR-EMP-00006'},
 {'name': 'HR-EMP-00010'},
 {'name': 'HR-EMP-00005'}
]

# with pluck
frappe.db.get_list('Employee', pluck='name')

# output
['HR-EMP-00008',
 'HR-EMP-00006',
 'HR-EMP-00010',
 'HR-EMP-00005'
]
```

Combining filters and other arguments:

```python
frappe.db.get_list('Task',
	filters={
		'status': 'Open'
	},
	fields=['subject', 'date'],
	order_by='date desc',
	start=10,
	page_length=20,
	as_list=True
)

# output
(('Update Branding and Design', '2019-09-04'),
('Missing Documentation', '2019-09-02'),
('Fundraiser for Foundation', '2019-09-03'))

# Tasks with date after 2019-09-08
frappe.db.get_list('Task', filters={
	'date': ['>', '2019-09-08']
})

# Tasks with date between 2020-04-01 and 2021-03-31 (both inclusive)
frappe.db.get_list('Task', filters=[[
	'date', 'between', ['2020-04-01', '2021-03-31']
]])

# Tasks with subject that contains "test"
frappe.db.get_list('Task', filters={
	'subject': ['like', '%test%']
})

# Count number of tasks grouped by status
frappe.db.get_list('Task',
	fields=['count(name) as count', 'status'],
	group_by='status'
)
# output
[{'count': 1, 'status': 'Working'},
 {'count': 2, 'status': 'Overdue'},
 {'count': 2, 'status': 'Open'},
 {'count': 1, 'status': 'Filed'},
 {'count': 20, 'status': 'Completed'},
 {'count': 1, 'status': 'Cancelled'}]
```

## frappe.db.get_all
`frappe.db.get_all(doctype, filters, or_filters, fields, order_by, group_by, start, page_length)`

- Also aliased to `frappe.get_all`

Same as `frappe.db.get_list` but will fetch all records without applying permissions.

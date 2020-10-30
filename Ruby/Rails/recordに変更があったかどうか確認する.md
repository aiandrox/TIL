## after_create/after_update より前の場合

```
will_save_change_to_attribute?
attribute_change_to_be_saved
attribute_in_database
changes_to_save
has_changes_to_save?
changed_attribute_names_to_save
attributes_in_database
```

## after_create/after_update より後の場合

```
saved_change_to_attribute?
saved_change_to_attribute
attribute_before_last_save
saved_changes
saved_changes?
saved_changes.keys
saved_changes.transform_values(&:first)
```

[Rails 5\.1 で attribute\_was, attribute\_change, attribute\_changed?, changed?, changed 等が DEPRECATION WARNING \- Qiita](https://qiita.com/htz/items/56798d53ec5988733fc6)

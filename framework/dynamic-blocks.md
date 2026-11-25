# Dynamic blocks

[Documentation home](../README.md) | [Framework](README.md) | [Meros Blocks](../packages/meros-blocks/README.md)

## Contents

- [Nested dynamic blocks](#nested-dynamic-blocks)
- [Inspector controls](#inspector-controls)
- [Repeater controls and testing](#repeater-controls-and-testing)

## Nested dynamic blocks

A Meros dynamic block can declare its direct parent with `$child->parent([$parent])`.
The parent gets a native block inserter and an editable area for its children.
By default, the inserter allows the dynamic blocks that declared that parent.
Use `$parent->allowedBlocks([...])` to set the permitted block names explicitly.

WordPress saves the nested blocks with the parent. To display the rendered
children on the front end, include `$args['content']` in the parent's
`renderCallback` output.

## Inspector controls

The attribute's `type` is its WordPress storage type. Use `control => true`
to infer a control from that type, or `control => ['type' => 'url']` to show
a labeled URL field for a `string` attribute. The field fits the inspector
sidebar. Paste a URL or type to search for an existing link using WordPress
suggestions. Set `placeholder` in the control options to override the default prompt.

Set `conditions` in the control options to show it only when other attributes
have the specified values:

```php
$this->attribute('form_id', [
    'type' => 'integer',
    'default' => 0,
    'control' => [
        'conditions' => ['use_current_form_id' => false],
    ],
]);
```

Conditions reference attributes defined on the same block and accept scalar
or null values. All entries must match, using strict equality (for example,
`false` is different from `0` or `'false'`). Updates toggle visibility immediately
without clearing the hidden attribute's value. Conditions affect only the
inspector, not front-end rendering or server-side validation.

## Repeater controls and testing

See [repeaters in dynamic block inspectors](forms.md#repeaters-in-dynamic-block-inspectors)
for control callbacks, row-edit forms, lookups and editor assets.

Test native WordPress block registration and rendering using the
[package developer integration workflow](testing/package-developers.md#block-registration-and-rendering).

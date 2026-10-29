Strukt Form
===

Rule-based form definition, validation, and error reporting, built on the Strukt
rules engine.

A form is described by its rules. There is no docblock parsing, no reflection,
and no request object to hand: give the form an array of submitted values and it
reports which fields failed and why.

## Install

```sh
composer require strukt/form
```

Requires PHP 8.2+ and `strukt/rules`.

## Defining a form

Extend `Strukt\Form\Form` and return rules keyed by field name. A field may use a
pipe-separated string or a list of rule tokens.

```php
use Strukt\Form\Form;

final class LoginForm extends Form
{
    public function rules(): array
    {
        return [
            'email' => 'required|email',
            'password' => 'required|min_length:8',
        ];
    }
}
```

```php
public function rules(): array
{
    return [
        'name' => 'required|string|min_length:2|max_length:50',
        'email' => ['required', 'email', 'max_length:120'],
        'age' => 'nullable|integer|min:18',
        'role' => 'required|in:admin,editor,viewer',
        'tags' => 'array|max_count:5',
        'bio' => 'nullable|max_length:500',
    ];
}
```

The full rule list, messages, and custom rules live in `strukt/rules`.

## Submitting and validating

```php
$form = new LoginForm();
$form->submit($_POST);

if ($form->validate() === false) {
    foreach ($form->errors() as $field => $messages) {
        // ['email' => ['Email must be a valid email address.']]
    }
}
```

Use `validateWith()` to submit and validate in one call:

```php
$form->validateWith([
    'email' => 'ada@example.com',
    'password' => 'analytical-engine',
]);

$form->validate();          // true
$form->hasErrors();         // false
$form->errors();            // []
$form->errorsFor('email');  // []
```

## Form API

| Method | Purpose |
| --- | --- |
| `fields()` | Field names in declaration order. |
| `rules()` | The rules definition. |
| `value($field, $default = null)` | One submitted value. |
| `setValue($field, $value)` | Set one value. |
| `setValues(array $values)` | Merge several values. |
| `values()` | Every submitted value. |
| `isSubmitted()` | Whether values have been submitted. |
| `submit(array $values = [])` | Merge values and mark as submitted. |
| `validate()` | Run the rules; true when everything passes. |
| `validateWith(array $values)` | Submit, then validate. |
| `hasErrors()` | Whether any field failed. |
| `errors()` | Failure messages keyed by field. |
| `errorsFor($field)` | Failure messages for one field. |
| `reset()` | Clear values, errors, and the submitted flag. |
| `validated(array $values)` | A fresh submitted form with no failures. |

`submit()`, `setValue()`, and `setValues()` accumulate. Call `reset()` first, or
use `validated()`, to start over.

```php
$fresh = $form->validated($otherSubmission);
```

## Messages

Override a message per rule, or for one field with `field.rule`. The field
specific override wins. An override is a complete message used as written and may
use `:field`, `:0`, `:1`, `:list`, and `:format`.

```php
public function messages(): array
{
    return [
        'email.required' => 'We need an email address to reach you.',
        'terms.required' => 'You must accept the terms to continue.',
        'min_length' => ':field is too short, use at least :0 characters.',
    ];
}
```

## Form-specific custom rules

Register a custom rule on a form-specific registry so the rest of the application
is unaffected.

```php
use Strukt\Rules\RuleRegistry;

final class ReservationForm extends Form
{
    protected function registry(): RuleRegistry
    {
        return RuleRegistry::create()->extend(
            'after',
            fn (string $field, array $params) => new AfterRule($field, $params[0]),
        );
    }

    public function rules(): array
    {
        return [
            'end' => 'required|date|after:start',
            'start' => 'required|date',
        ];
    }
}
```

## Helpers

```php
form(LoginForm::class, ['email' => $email, 'password' => $password]);
form(LoginForm::class, $values, submit: false);

form_check(
    ['age' => 12],
    ['age' => 'required|integer|min:18'],
);
```

`form()` creates and submits a form instance. `form_check()` validates a plain
array against a rules definition without a form class.

## Integration

The router can attach a form with `form()` on a route. It submits the request
values, validates before the action runs, and returns a 422 JSON payload with
field errors when validation fails.

```php
$router->post('/login', fn (
    Strukt\Contract\FormInterface $form,
) => 'hi '.$form->value('email'))->form(LoginForm::class);
```

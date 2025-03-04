---
theme: uncover
paginate: true
class:
  - lead
  - invert
size: 16:9
style: |
  .small-text {
    font-size: 0.75rem;
  }
footer: "Philip Norton [hashbangcode.com](https://www.hashbangcode.com) [@hashbangcode](https://twitter.com/hashbangcode) [@philipnorton42](https://twitter.com/philipnorton42)"
marp: true

---

# New Features In<br>Drupal 10.2.0

<!-- Live demos abound! Wish me luck! -->
---

## Drupal 10.2.0 Is A Minor Release

- This is a minor release and so should not break backwards compatability.
- You should still test your sites though!

---


# Interface Improvements

---

## Media Revisions

Media entities now have a new tab for viewing, reverting and deleting revisions.

![height:350px](../src/assets/images/media_revisions.png)

---

## Taxonomy Add Child Button

Easier to add children to taxonomy terms.

![](../src/assets/images/taxonomy_add_child.png)

--- 

## Filename Sanitization

Is now configurable.

![height:400px](../src/assets/images/filename_sanitization.png)


---
<!-- _footer: '' -->
## Block Visibility

Can now use status code

![height:350px](../src/assets/images/block_visibility.png)

<p class="small-text">(Note: The <strong>Block Layout</strong> and <strong>Block Types</strong> area have already been moved under the <strong>Structure</strong> menu).</p>

---

## Field Creation

New interface!

![height:400px](../src/assets/images/new_field.png)

---

## Field Storage Form

The field storage form is now a sub form of the field configuration form.

![height:350px](../src/assets/images/field_storage_edit.png)

---

## Login And Password Reset Froms

- Now include autocomplete attributes. 
- This gives better hints to browsers as to what can be filled in.
- If, for some reason, you have disabled this then you'll need to check that it still works.

---

# CKEditor 5 Improvements

---

## Tables

New formatting for tables and table cells when used with full HTML text formats.

![height:350px](../src/assets/images/ckeditor_table.png)

---

## Show Blocks

Reintroduced into CKEditor 5.

![height:350px](../src/assets/images/ckeditor_codeblocks.png)

--- 

## HTML Comments

HTML comments are also now correctly preserved in CKEditor.

```html
<p>Text.</p>
<!-- Monkey -->
<p>Text.</p>
```

---

# Code Improvements

---
<!-- _footer: "" -->
## Plugins Can Now Use PHP Attributes

This will replace the existing doctrine annotations.

Before:
```php
/**
 * Provides a block to display the local actions.
 *
 * @Block(
 *   id = "local_actions_block",
 *   admin_label = @Translation("Primary admin actions")
 * )
 */
class LocalActionsBlock extends BlockBase implements ContainerFactoryPluginInterface {
```

---

After:

```php
/**
 * Provides a block to display the local actions.
 */
#[Block(
  id: "local_actions_block",
  admin_label: new TranslatableMarkup("Primary admin actions")
)]
class LocalActionsBlock extends BlockBase implements ContainerFactoryPluginInterface {
```

Existing plugins should start migrating over to PHP Attributes.

---

## Autoconfiguration Of Event Subscribers

By adding the autoconfigure flag you don't need to add the tag of "event_subscriber".

```yaml
services:
  _defaults:
    autoconfigure: true
  response_generator_subscriber:
    class: Drupal\Core\EventSubscriber\ResponseGeneratorSubscriber
```

---

## Controller Auto-wiring

- Controllers no longer require a create() method to inject dependencies. Services are interred from the interfaces in the constructor alone.
- If this is not possible then the `#[Autowire]` attribute can be used to specify the service.

```php
  public function __construct(
    #[Autowire(service: 'cache.default')]
    protected CacheBackendInterface $cache
 ) {}
 ```

--- 

## Filter System Now Outputs HTML5

Previsouly, the filtering system outputed XHTML.

Before:
```html
<p>
  Example text
  <br />
  <img src="sites/defaut/files/image.jpg" />
</p>
```

---

After:
```html
<p>
  Example text
  <br>
  <img src="sites/defaut/files/image.jpg">
</p>
```

- In most cases this shouldn't have any effect.
- Anything looking for very specific XHTML output may need to be updated.

---

## Configuration Targets Can Now Be Set

Inform Drupal to use a config item as the `#default_value` without setting it.

```php
   $form['registration_cancellation']['user_email_verification'] = [
      '#type' => 'checkbox',
      '#title' => $this->t('Require email verification when a visitor creates an account'),
      '#config_target' => 'user.settings:verify_mail',
    ];
```

<!-- 
- Part of the new config validation system.
- The form must pass through the parent buildForm() method for this to work.
-->

---

## DeprecationHelper

New class to aid modules supporting multiple versions of core.

```php
$result = DeprecationHelper::backwardsCompatibleCall(
        currentVersion: \Drupal::VERSION,
        deprecatedVersion: '10.3',
        currentCallable: fn() => Role::loadMultiple(),
        deprecatedCallable: fn() => user_roles(),
);
```

<!-- 
- If the current version of Drupal is below 10.3 then run user_roles().
- If the current version of Drupal is 10.3 or above then run Role::loadMultiple().
-->

---

## Schema Strictly Validated In Tests

- Assures that your configuration schema is valid when running tests.
- May cause some modules to start throwing errors during tests when they previsouly didn't.
- Effects kernel and functional tests.

---
<!-- _footer: "" -->

## Collection Permission

<p class="small-text">Entities may provide a collection_permission to access the entity overview list.</p>

```php
/**
 * @ContentEntityType(
 *   ...
 *   handlers = {
 *     "list_builder" = "Drupal\my_awesome_module\MyAwesomeListBuilder",
 *   ...
 *   },
 *   links = {
 *     "collection" = "/admin/structure/my-awesome-entities",
 *   },
 *   admin_permission = "administer my awesome entities",
 *   collection_permission = "access the awesome entity overview",
```

---


# Experimental Module Changes

---

## Announcements Feed

Promoted to stable.

![height:350px](../src/assets/images/announcements.png)

---


## Experimental Help Topics

Merged into the help module.


![height:350px](../src/assets/images/help_topics.png)

---

# Permission Changes

---

## Rebuild Node Permissions

- This action now requires the `administer nodes` permission.
- Previsouly, only the `access administration pages` option was required.

---

# Performance Improvements

---
<!-- _footer: '' -->
## VariationCache API

- The VariationCache module has been moved into core.
- Allows you to use cache contexts with any cache backend.

```php
$cacheable_metadata = CacheableMetadata::createFromObject($response->getCacheableMetadata());
$this->cache->set(
  ['response'],
  $response,
  $cacheable_metadata->addCacheContexts($this->cacheContexts),
  (new CacheableMetadata())->setCacheContexts($this->cacheContexts)
);
```

- The Group module uses VariationCache.

---

## webp Images

- webp image support is now in core.
- Drupal core shipped image styles will now convert to webp.

![height:350px](../src/assets/images/webp.png)

---

## content-length Header

- Drupal now sets a content-length header for most responses.
- This includes all non-streamed responses (so, anything not using BigPipe).
- Allows services implementing `DestructableInterface` to run after the page has been delivered to the client.
- May need to enable output buffering by default in PHP.

---

## PHP Fibers

- Big Pipe rendering now makes use of PHP Fibers.
- Read more: [#! Code : Fibers In PHP 8.1](https://www.hashbangcode.com/article/fibers-php-81)

---

# And Finally...

---

## PHP Version

Drupal now supports PHP 8.3 and recommends at least PHP 8.2.

---

## Symfony Components

Symfony components havebeen updated to version 6.4.

---

## Symfony Mailer

- The Symfony Mailer component has been added to Drupa lcore.
- Ultimate aim is to replace the existing mail system in a future release.

---

## Upgrading From Drupal 9

- Make sure you update to Drupal 9.4.4 before updating to Drupal 10.

- Remember that Drupal 9 is currently end of life and will receive no security updates.

---

## Resources

[Drupal 10.2.0-rc1](https://www.drupal.org/project/drupal/releases/10.2.0-rc1)

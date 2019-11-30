# TYPO3 Extension ``routes``

## 1. Features

- Provides an opportunity to call any extbase actions by url
- Route definition based on YAML
- Contains View Helper for building routes by name
- All routes could be strongly cached

## 2. Usage

### 1) Installation

#### Installation using Composer

The recommended way to install the extension is by using [Composer][2]. In your Composer based TYPO3 project root, just do `composer require lms/routes`.

#### Installation as extension from TYPO3 Extension Repository (TER)

Will be possible in the future.

### 2) Minimal setup

1) Include the static TypoScript of the extension.
2) Create routes file under your extension **ext/my_extension/Configuration/Routes.yml**
3) Define your route in the **Routes.yml**

## 3. Examples

### 1. Simple route without parameters
```yaml
# ext/example/Configuration/Routes.yml

client_list:
  path:       clients
  controller: Vendor\Example\Controller\ClientController::index
```
This is minimum that should be provided for any route definition
- client_list
    - It is a route name. This name could be used for building route path in the Fluid area.
    - Required: **yes**
- path
    - Represents the route itself
    - Required: **yes**
- controller
    - Defines the endpoint Extbase Action, that will be triggered by defined path
    - Required: **yes**

```php
    /**
     * Show all clients
     */
    public function indexAction(): void
    {
        $this->view->assign('clients', $clientRepository->findAll());
    }
```


### 2. Route with GET parameters
```yaml
# ext/example/Configuration/Routes.yml

client_edit:
  path:         clients/{client}
  controller:   Vendor\Extension\Controller\ClientController::edit
  requirements:
    client:     \d+
```

- requirements
    - Defines validation rules for parameter
    - Required: **no**
    - Notice: Parameter names in path and inside requirements should have **same** names.

```php
    /**
     * Modify client information
     *
     * @param \Vendor\Extension\Domain\Model\Client $client
     */
    public function editAction(Client $client): void
    {
        $this->view->assign('client', $client);
    }
```

### 3. Route with POST parameters
```yaml
# ext/example/Configuration/Routes.yml

client_create:
  path:          clients
  controller:    Vendor\Extension\Controller\ClientController::create
  defaults:
    title:
    description:
```
In situation when data is not present in the request URL, but anyways passed thought POST, the names of the passed
parameters should be present in **defaults** section.

```php
    /**
     * Create a brand new client based on passed data
     *
     * @param string $title
     * @param string $description
     */
    public function createAction(string $title, string $description): void
    {
        $this->view->setVariablesToRender(['client']);

        $client = $clientRepository->create($title, $description);

        $this->view->assign('client', $client);
    }
```

### 4. Define Route Method
```yaml
# ext/example/Configuration/Routes.yml

client_remove:
  path:         clients/{client}
  controller:   Vendor\Extension\Controller\ClientController::delete
  methods:      [DELETE]
  requirements:
    client:     \d+
```

- methods
    - Action will be called only if request method is matched.
    - Required: **no**
    - Options: **GET** **POST** **DELETE** **HEAD**
```php
    /**
     * Remove existing client
     *
     * @param \Vendor\Extension\Domain\Model\Client $client
     */
    public function deleteAction(Client $client): void
    {
        $clientRepository->remove($client);

        $this->addFlashMessage('Client has been removed.');

        $this->redirect('index');
    }
```

### 5. Define Route Scheme
```yaml
# ext/example/Configuration/Routes.yml

client_list:
  path:       clients
  controller: Vendor\Example\Controller\ClientController::index
  schemes:    [https]
```

- schemes
    - Route will be called only if scheme is matched
    - Required: **no**
    - Options: **http** **https**

### 6. Route View Helper
```html
{namespace route = LMS\Routes\ViewHelpers}

<a href="{route:makeSlug( for: 'client_edit', with: {client: 2} )}">
    Edit
</a>
```

### 7. Csrf Token
- HTML Scope

```html
{namespace route = LMS\Routes\ViewHelpers}

{route:csrfToken()}
```
- JavaScript Scope:

```javascript
document.head.querySelector('meta[name="x-csrf-token"]').content
```

### 8. Route Middleware

```yaml
# ext/example/Configuration/Routes.yml

client_list:
  path:         clients
  controller:   Vendor\Extension\Controller\ClientController::index
  options:
    middleware:
      - auth
      - Vendor\Extension\Middleware\Api\MyCustomMiddleware
```
- middleware
    - Action will be called only if middleware passed
    - Required: **no**
    - Variants:
        * LMS\Routes\Middleware\Api\Authenticate
        * LMS\Routes\Middleware\Api\VerifyCsrfToken
        * Vendor\Extension\Middleware\Api\MyCustomMiddleware
        * auth just sugar that combines (Authenticate and VerifyCsrfToken)

- Middleware doesn't need to follow any contracts, here's an example:
```php
    namespace Vendor\Extension\Middleware\Api;

    use Symfony\Component\Routing\Exception\MethodNotAllowedException;

    class MyCustomMiddleware
    {
        /**
         * @param array $arguments
         */
        public function process(array $arguments): void
        {
            $isOk = true;

            if ($isOk) {
                return;
            }

           throw new MethodNotAllowedException([], 'Denied!');
        }
    }
```

[1]: https://docs.typo3.org/typo3cms/extensions/routes/
[2]: https://getcomposer.org/

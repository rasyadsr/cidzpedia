---
sidebar_position: 1
---

# The Basics

Merupakan rules atau standarisasi yang digunakan ketika memabangun aplikasi berbasif NAF

## Perlu di ingat

Setiap apapun itu baik routing, controller, view harus kita registrasikan

- Routing

```php title="src/Neuron/namaModule/Config.php"
'router' => array(
        'routes' => array(
            /* Konfigurasi routing modul Attendance */
            'attendance' => array(
                'type' => 'Zend\Mvc\Router\Http\Literal',
                'options' => array(
                    'route' => '/attendance',
                    'defaults' => array(
                        '__NAMESPACE__' => 'Neuron\Attendance\Controller', /* Set default routing fallback */
                        'controller' => 'Index',
                        'action' => 'index',
                    ),
                ),
                'may_terminate' => true,
                'child_routes' => array(
                    'default' => array(
                        'type' => 'Segment',
                        'options' => array(
                            'route' => '/[:controller[/:action]]', /* Set konfigurasi auto routing, selain default */
                            'constraints' => array(
                                'controller' => '[a-zA-Z][a-zA-Z0-9_-]*',
                                'action' => '[a-zA-Z][a-zA-Z0-9_-]*',
                            ),
                            'defaults' => array(
                            ),
                        ),
                    ),
                ),
            ),
            'widget_attendance' => array(
                'type' => 'Zend\Mvc\Router\Http\Literal',
                'options' => array(
                    'route' => '/widget_attendance',
                    'defaults' => array(
                        '__NAMESPACE__' => 'Neuron\Attendance\Controller\Widget', /* Set default routing fallback */
                        'controller' => 'Attendance',
                        'action' => 'index',
                    ),
                ),
                'may_terminate' => true,
                'child_routes' => array(
                    'default' => array(
                        'type' => 'Segment',
                        'options' => array(
                            'route' => '/[:controller[/:action]]', /* Set konfigurasi auto routing, selain default */
                            'constraints' => array(
                                'controller' => '[a-zA-Z][a-zA-Z0-9_-]*',
                                'action' => '[a-zA-Z][a-zA-Z0-9_-]*',
                            ),
                            'defaults' => array(
                            ),
                        ),
                    ),
                ),
            ),
            'api_attendance' => array(
                'type' => 'Zend\Mvc\Router\Http\Literal',
                'options' => array(
                    'route' => '/api_attendance',
                    'defaults' => array(
                        '__NAMESPACE__' => 'Neuron\Attendance\Controller\Api', /* Set default routing fallback */
                        'controller' => 'Attendance',
                        'action' => 'index',
                    ),
                ),
                'may_terminate' => true,
                'child_routes' => array(
                    'default' => array(
                        'type' => 'Segment',
                        'options' => array(
                            'route' => '/[:controller[/:action]]', /* Set konfigurasi auto routing, selain default */
                            'constraints' => array(
                                'controller' => '[a-zA-Z][a-zA-Z0-9_-]*',
                                'action' => '[a-zA-Z][a-zA-Z0-9_-]*',
                            ),
                            'defaults' => array(),
                        ),
                    ),
                ),
            ),
        ),
```

- Controller

```php title="src/Neuron/namaModule/Config.php"
     'controllers' => array(
        'invokables' => array(
            /* controllers for Attendance */
            'Neuron\Attendance\Controller\Index' => 'Neuron\Attendance\Controller\Index',
            'Neuron\Attendance\Controller\Form' => 'Neuron\Attendance\Controller\Form',
        )
    )
```

- View

```php title="src/Neuron/namaModule/Config.php"
    'view_manager' => array(
        'template_map' => array(
            'neuron/task/layout/form' => __DIR__ . '/View/Form.phtml',
        )
    );
```

**Note :**

- Jika ngodingnya di aplikasi maka melakukan registrasinya di `root-project\application\Neuron\Application{ namaAplikasi }\config\module.config.php`

- `$this->basePath()` itu hanya sampai `http://localhost/root-project/public/` maka kita harus menambahkan route namaAplikasi nya contohnya `<?php echo $this->basePath() ?>/{namaModule}/{namaController}/{namaMethod}`

## Priority

- Ketika kita set env ke dev, maka aplikasi akan membaca dari file `*.dev.php` ( yang ada dev nya ) namun jika tidak ada akan memanggil `*.php`

## Request LifeCycle

1. Route ( Controller dan Method default )
2. Controller
3. Method
4. Jika ada hubungannya dengan database maka akan menggunakan Model
5. Model
6. Storage
7. Mysql / Postgree / Oracle ( tergantung database yang di pilih )

## Folder

- Config aplikasi di simpan di module.config.php
- Controller di simpan di folder controller
- View di simpan di folder view

### Controller

- setiap controller yang akan mereturn view, di method nya wajib menggunakan akhiran `Action()` misal `indexAction()`

### View

- setiap controller yang akan mereturn view harus membuat `function getView($view, $variables = [])` dengan access modifier protected
- di dalam method `getView()` kita setting template dan prefix
- `$layout = "neuron/{ namaAplikasi }/layout/form";` ( layout default )
- `$prefix = "neuron/{ namaAplikasi }/namaController";` ( prefix adalah sebuah awalan yang nantinya akan di daftarkan di view_manager )
- `$this->layout($layout);` ( digunakan untuk set layout )
- `$view = new Zend\View\Model\ViewModel($variables);` ( standard class untuk return view dengan data )
- `$view->setTemplate($prefix .'/'. $view);` ( digunakan untuk set prefix dengan view nya )
- `return $view` ( mereturn view yang sebelumnya sudah kita set )

### Model / Database

- Model di simpan di folder model
- 1 tabel di database memiliki 2 model, yaitu **Singular**(nama Model tanpa 's') dan **prular**(nama Model dengan 's')
- Model Singular digunakan untuk data yang singleRow
- Model prular digunakan untuk data yang manyRow
- class yang `Singular extends \Neuron\Generic\Entity\Mapper`

```php title="src/namaModule/Attendance.php"
    class Attendance extends \Neuron\Generic\Entity\Mapper {}
```

- class yang `Prular extends \Neuron\Generic\Set`

```php title="src/namaModule/Attendances.php"
    class Attendances extends \Neuron\Generic\Set {}
```

- setiap model baik singular maupun prular wajib memiliki `construct()` yang berisi parameter dari object `\Neuron\Db\Storage` dan `$data = []` (fitur yamg bisa digunakan mirip seperti mapper)
- 1 tabel di database memiliki 1 folder di dalam folder model ( untuk storage ) yang di beri nama sesuai nama tabel nya`
- Folder yang do peruntukan untuk storage tadi di dalam nya memiliki 1 folder Storage dan 1 file Storage.php
- Folder Storage berisi interface Skeleton.php dan class dari driver database seperti Mysql.php, Postgree.php, dll yang di mana class tersebut harus implements Skeleton
- File `Storage.php` berisi class Storage yang memiliki constanta LOADBY_ID = 0, LIST_ALL = 0, dan static function factory yang berisi parameter adapter dan config yang di gunakan untuk mengecek db_driver apa yang di pakai oleh aplikasi kita
- `$this->__()` // ini untuk mengirim string -` $this->_()` // ini untuk mengirim array

## Standarisasi Return API

- Query di lakukan di file db_driver nya seperti `Mysql.php`, `Postgree.php`, dll
- Logic Query di lakukan di Model nya
- `Neuron\Generic\Result` merupakan standard class untuk return value yang nanti akan di return dalam function `$this->getREST()->getResponse($result)`

```php
$result = new \Neuron\Generic\Result();
$result->code = 0 // (0 berarti success, selain itu gagal)
$result->info = 'success',
$result->data = $data //  ( data yang akan di return )
$this->getREST()->getResponse($result)
```

## Menangkap parameter

```php
- $_GET = $this->getRequest()->getQuery()
- $_POST = $this->getRequest()->getPost()
- $_FILES = $this->getRequest()->getFiles()
```

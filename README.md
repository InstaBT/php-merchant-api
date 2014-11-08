PHP Example for the InstaBT Payments API
========================================


```php
<?php
$serverURL   = 'https://api.instabt.com';
$apiKey      = 'ENTER_YOUR_API_KEY_HERE';
$apiSecret   = 'ENTER_YOUR_API_SECRET_HERE';

// Microseconds since epoch
function usecTime() {
    list($usec, $sec) = explode(' ', microtime());
    $usec = substr($usec, 2, 6);
    return intval($sec.$usec);
}

$endpoint = '/v1/create_order';
$url = $serverURL . $endpoint;

$parameters= array();
$parameters['amount'] = 9.95;
$parameters['currency'] = 'CAD';
$parameters['data'] = 'order_id=1773';
$parameters['nonce']    = usecTime();
$data = http_build_query($parameters);

$httpHeaders = array(
    'Api-Key: '   . $apiKey,
    'Api-Sign:'   . base64_encode(hash_hmac('sha512', $endpoint . chr(0) . $data, $apiSecret)),
);

// Initialize the PHP curl agent
$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, $url);
curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
curl_setopt($ch, CURLOPT_FAILONERROR, true);
curl_setopt($ch, CURLOPT_HTTPHEADER, $httpHeaders);
curl_setopt($ch, CURLOPT_POSTFIELDS, $data);

$result = curl_exec($ch);
if ($result === false)
  throw new Exception ("curl Error: " . curl_error($ch));

$http_status = curl_getinfo($ch, CURLINFO_HTTP_CODE);
if ($http_status != 200)
    throw new Exception("Request Failed. http status: " . $http_status);
  curl_close($ch);

// Trim any whitespace from the front and end of the string and decode it
$result = json_decode(trim($result));
if (($error = get_json_error()) !== false) {
    throw new Exception("json_decode failed: " . $error);
}

$invoice_url = $result.Url;

// Redirect user to invoice
header('Location: '.$invoice_url);
?>
```

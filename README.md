# faker-image-issue
this is an issue where faker won't generate images to your project

to fix this, just replace the file on "vendor\fakerphp\faker\src\Faker\Provider\Image.php" with the file on top or edit by yourself:

public const BASE_URL = 'https://placehold.jp'; // change the url, the old one isn't working anymore

at the function:
        // save file
        if (function_exists('curl_exec')) {
after the line: 
curl_setopt($ch, CURLOPT_FILE, $fp);
and before the line:
$success = curl_exec($ch) && curl_getinfo($ch, CURLINFO_HTTP_CODE) === 200;

add those two lines:
curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, false);
curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);

your code will be like that:

   // save file
        if (function_exists('curl_exec')) {
            // use cURL
            $fp = fopen($filepath, 'w');
            $ch = curl_init($url);
            curl_setopt($ch, CURLOPT_FILE, $fp);
            curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, false); //new line
            curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false); //new line
            $success = curl_exec($ch) && curl_getinfo($ch, CURLINFO_HTTP_CODE) === 200;
            fclose($fp);
            curl_close($ch);

            if (!$success) {
                unlink($filepath);

                // could not contact the distant URL or HTTP error - fail silently.
                return false;
            }
        } elseif (ini_get('allow_url_fopen')) {
            // use remote fopen() via copy()
            $success = copy($url, $filepath);

            if (!$success) {
                // could not contact the distant URL or HTTP error - fail silently.
                return false;
            }
        } else {
            return new \RuntimeException('The image formatter downloads an image from a remote HTTP server. Therefore, it requires that PHP can request remote hosts, either via cURL or fopen()');
        }

        return $fullPath ? $filepath : $filename;
    }

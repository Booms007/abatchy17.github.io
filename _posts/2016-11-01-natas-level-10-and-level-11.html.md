Part two [here](http://abatchy.blogspot.com/2016/10/natas-
level-6-level-9.html).  
  

That's a lot of code! Let's see what it does. Best way is to follow the code
execution, so functions will be ignored till they're actually being called.

  

    
    
     1  
     2  
     3  
     4  
     5  
     6  
     7  
     8  
     9  
    10  
    11  
    12  
    13  
    14  
    15  
    16  
    17  
    18  
    19  
    20  
    21  
    22  
    23  
    24  
    25  
    26  
    27  
    28  
    29  
    30  
    31  
    32  
    33  
    34  
    35  
    36  
    37  
    38  
    39  
    40  
    41  
    42  
    43  
    44  
    45  
    46  
    47  
    48  
    49

|

    
    
    <?  
      
    $defaultdata = array( "showpassword"=>"no", "bgcolor"=>"#ffffff");  
      
    function xor_encrypt($in) {  
        $key = '<censored>';  
        $text = $in;  
        $outText = '';  
      
        // Iterate through each character  
        for($i=0;$i<strlen($text);$i++) {  
        $outText .= $text[$i] ^ $key[$i % strlen($key)];  
        }  
      
        return $outText;  
    }  
      
    function loadData($def) {  
        global $_COOKIE;  
        $mydata = $def;  
        if(array_key_exists("data", $_COOKIE)) {  
        $tempdata = json_decode(xor_encrypt(base64_decode($_COOKIE["data"])), true);  
        if(is_array($tempdata) && array_key_exists("showpassword", $tempdata) && array_key_exists("bgcolor", $tempdata)) {  
            if (preg_match('/^#(?:[a-f\d]{6})$/i', $tempdata['bgcolor'])) {  
            $mydata['showpassword'] = $tempdata['showpassword'];  
            $mydata['bgcolor'] = $tempdata['bgcolor'];  
            }  
        }  
        }  
        return $mydata;  
    }  
      
    function saveData($d) {  
        setcookie("data", base64_encode(xor_encrypt(json_encode($d))));  
    }  
      
    $data = loadData($defaultdata);  
      
    if(array_key_exists("bgcolor",$_REQUEST)) {  
        if (preg_match('/^#(?:[a-f\d]{6})$/i', $_REQUEST['bgcolor'])) {  
            $data['bgcolor'] = $_REQUEST['bgcolor'];  
        }  
    }  
      
    saveData($data);  
      
      
      
    ?>  
      
  
---|---  
  
  
What does this code do?  

  * $defaultdata: Array containing two values, _showpassword _string and _bgcolor _contaning the div background value.
  * **Line 37**: $data is loaded via the loadData function on line 18, a temporary array $mydata is used to store the default values ("no", "#ffffff"). If the cookie sent in the HTTP request contains a field called data, it tries to decode it as a JSON object. The value has first to be decoded in base 64, then XORed by calling xor_encrypt. If the JSON is a valid array and contains both _shownopassword / bgcolor_**  **fields, it's then written to $mydata, **thus overriding the default values**.
  * **Line 39**: If bgcolor key is passed in the request and matches the regex (for HEX colors string), it's then saved in the $data array.
  * **Line 45**:  saveData stores the cookie with the updated values. 

  

    
    
     1  
     2  
     3  
     4  
     5  
     6  
     7  
     8  
     9  
    10  
    11  
    12  
    13  
    14  
    15  
    16  
    17  
    18  
    19

|

    
    
    <h1>natas11</h1>  
    <div id="content">  
    <body style="background: <?=$data['bgcolor']?>;">  
    Cookies are protected with XOR encryption<br/><br/>  
      
    <?  
    if($data["showpassword"] == "yes") {  
        print "The password for natas12 is <censored><br>";  
    }  
      
    ?>  
      
    <form>  
    Background color: <input name=bgcolor value="<?=$data['bgcolor']?>">  
    <input type=submit value="Set color">  
    </form>  
      
    <div id="viewsource"><a href="index-source.html">View sourcecode</a></div>  
    </div>  
      
  
---|---  
  
  *  HTML is then generated and data is loaded from the array $defaultdata. 

One important thing to know about XOR encoding is that it's vulnerable to
[known-plaintext attack](https://en.wikipedia.org/wiki/Known-
plaintext_attack).  
**  
****_plaintext_ ![\\oplus ](https://wikimedia.org/api/rest_v1/media/math/render/svg/8b16e2bdaefee9eed86d866e6eba3ac47c710f60) _key_ = _cyphertext_**  
**_plaintext_ ![\\oplus ](https://wikimedia.org/api/rest_v1/media/math/render/svg/8b16e2bdaefee9eed86d866e6eba3ac47c710f60) _ciphertext_ = _key_** ⊕ {\displaystyle \oplus }   
  

#### 1\. Preprocessing data

####  

What we want is to XOR our plain text ($_data) _with the cyphertext_
($_COOKIE["data"])_, but we'll need to undo any preprocessing before XORing
them.  

  1. Cyphertext needs to be decoded: _base64_decode($_COOKIE["data"]))_
  2. Default data needs to be json encoded:_json_encode(array( "showpassword"=&gt;"no", "bgcolor"=&gt;"#ffffff"));_

#### 2\. Getting the key

####  

####

Normally, a long repetitive key is used for XORing, we want to figure out the
key, so we can re-encode the data and submit the manipulated cookie, setting
the **showpassword **parameter to "yes".

  

We'll be reusing/refactoring some PHP code:

  

    
    
     1  
     2  
     3  
     4  
     5  
     6  
     7  
     8  
     9  
    10  
    11  
    12  
    13  
    14  
    15  
    16  
    17  
    18  
    19  
    20

|

    
    
    <?php  
      
    $cookie = "ClVLIh4ASCsCBE8lAxMacFMZV2hdVVotEhhUJQNVAmhSEV4sFxFeaAw=";  
      
    function xor_encrypt($in) {  
        $key = json_encode(array( "showpassword"=>"no", "bgcolor"=>"#ffffff"));  
        $text = $in;  
        $outText = '';  
      
        // Iterate through each character  
        for($i=0;$i<strlen($text);$i++) {  
        $outText .= $text[$i] ^ $key[$i % strlen($key)];  
        }  
      
        return $outText;  
    }  
      
    echo xor_encrypt(base64_decode($cookie));  
      
    ?>  
      
  
---|---  
  
  

Next, we'll execute this code.

  

    
    
    1  
    2

|

    
    
    $ php -f p.php  
    qw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jq  
      
  
---|---  
  
  
We got our key!  
  

#### 3\. Encoding the new data

####  

For this step I had to append a couple more strings of "qw8J", using the key
provided in the previous snipped will not generate the correct string.  
  

    
    
     1  
     2  
     3  
     4  
     5  
     6  
     7  
     8  
     9  
    10  
    11  
    12  
    13  
    14  
    15  
    16  
    17  
    18  
    19  
    20

|

    
    
    <?php  
      
    $data = array( "showpassword"=>"yes", "bgcolor"=>"#ffffff");  
      
    function xor_encrypt($in) {  
        $key = 'qw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jq';  
        $text = $in;  
        $outText = '';  
      
        // Iterate through each character  
        for($i=0;$i<strlen($text);$i++) {  
        $outText .= $text[$i] ^ $key[$i % strlen($key)];  
        }  
      
        return $outText;  
    }  
      
    echo base64_encode(xor_encrypt(json_encode($data)));  
      
    ?>  
      
  
---|---  
  
  
  

    
    
    1  
    2

|

    
    
    $ php -f p.php  
    ClVLIh4ASCsCBE8lAxMacFMOXTlTWxooFhRXJh4FGnBTVF4sFxFeLFMK  
      
  
---|---  
  
  

#### 4\. Submitting the new cookie and getting the password

 In your browser's javascript console, edit the data cookie by writing:

**document.cookie="data=ClVLIh4ASCsCBE8lAxMacFMOXTlTWxooFhRXJh4FGnBTVF4sFxFeLFMK"**

  

Refresh the page and you got the password.** **

![](data:image/png;base64,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)

Password is: EDXp0pS26wLKHZy1rDBPUZk0RKfLGIR3

  

_Currently listening to [Violet Cold - Drowned In The Lights
](https://www.youtube.com/watch?v=o56YaB-Isck)_

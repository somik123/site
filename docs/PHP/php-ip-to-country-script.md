---
id: 80
title: PHP IP to Country Script
type: post
date: 2019-10-19 22:42:35
lastmod: 2023-04-01 23:53:42
categories:
- PHP
---


This is a simple script to translate IP to country. It is usefull when you want to know where your traffic is coming from or perform specific actions to traffic from specific countries.




Do take note that users using proxies or VPN can circumvent this and give a false country.







The free database is downloaded from ip2location and is stored as a sqlite database so your PHP must be installed with sqlite support.





```php

<?php

/**
* Author: Somik Khan
* @copyright 2019
* Permissions: Editing or non-commercial usage
* Prohibition: Sale or commercial usage.
* Updated script: https://somik.org/2019/10/19/php-ip-to-country-script/
*/

class ip2country_lite {
    // Actual DB file
    var $db_file = "./db/ip2location.sqlite";
    
    // Temporary file for IP database zip
    var $tmp_zip = "./db/tmp.zip";    
    
    // Link to ip2loc download
    var $link = "https://download.ip2location.com/lite/IP2LOCATION-LITE-DB1.CSV.ZIP";
    
    function __construct(){
        if(!file_exists($this->db_file)){
            $db_path = basename($this->db_file);
            if(!file_exists($db_path)){
                if(!mkdir($db_path,0755,TRUE)){
                    die("Cannot create \"db\" directory. Please create manually and ensure it is writable by php.");
                }
                else{
                    $this->update();
                }
            }
            else{
                $this->update();
            }
        }
    }
    
    function update(){
        // Get the CSV.ZIP from ip2loc
        $zip_contents = file_get_contents($this->link);
        file_put_contents($this->tmp_zip, $zip_contents);
        // Load the CSV into variable
        $csv_data = file_get_contents("zip://{$this->tmp_zip}#IP2LOCATION-LITE-DB1.CSV");
        // Delete the temp file
        unlink($this->tmp_zip);

        // Process the CSV into array. the format is "18939904","19005439","JP","Japan"
        preg_match_all('#"(\d*?)","(\d*?)","(.*?)","(.*?)"#',$csv_data,$arr_data);

        // Count the number of array items
        $count = count($arr_data[0]);
        // Remove old DB file
        @unlink($this->db_file);
        // Create a new DB file for writing
        $db = new SQLite3($this->db_file, SQLITE3_OPEN_CREATE | SQLITE3_OPEN_READWRITE);
        $db->busyTimeout(5000);

        // Create the table structures
        $db->query('CREATE TABLE IF NOT EXISTS "data" (
                        "start" INTEGER,
                        "end" INTEGER,
                        "country2" VARCHAR,
                        "country" VARCHAR
                    )');

        // Prepare DB insert statemetns but dont write to disk yet
        $db->exec('BEGIN');
        for($i=0;$i<$count;$i++){
            // Add the contents into DB still in memory
            $db->query('INSERT INTO "data" ("start", "end", "country2", "country")
                        VALUES ('.$arr_data[1][$i].', '.$arr_data[2][$i].', "'.$arr_data[3][$i].'", "'.$arr_data[4][$i].'" )');
        }
        // Write changes to DB in disk
        $db->exec('COMMIT');
        // Close the DB and complete.
        $db->close();
        
        echo "Updated";
    }
    
    function getCountry($ip){
        if($ip && file_exists($this->db_file)){
            // Check the IP is provided
            if(preg_match("/^(\d{1,3})\.(\d{1,3})\.(\d{1,3})\.(\d{1,3})$/",$ip,$parts)){
                // Convert IP to number by multiplying
                $ip_num = ( pow(256,3) * $parts[1] ) + ( pow(256,2) * $parts[2] ) + ( pow(256,1) * $parts[3] ) + ( pow(256,0) * $parts[4] );
                
                // Load the database and find the IP number
                $db = new SQLite3($this->db_file, SQLITE3_OPEN_READONLY);
                $db->busyTimeout(5000);
                $row = $db->querySingle('SELECT * FROM "data" WHERE "start" < '.$ip_num.' AND "end" > '.$ip_num.' ',1);
                $db->close();
                
                // Output the country
                echo json_encode( array($ip, $row['country'], $row['country2'] ) );
            }
        }
    }
}


// ----------
// Usage
// ----------

$ip = $_REQUEST['ip'];
if($ip){
    $ip2c = new ip2country_lite;
    $ip2c->getCountry($ip);
}
elseif(isset($_REQUEST['update'])){
    $ip2c = new ip2country_lite;
    $ip2c->update();
}


```



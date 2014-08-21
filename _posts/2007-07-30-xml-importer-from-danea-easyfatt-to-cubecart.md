---
layout: post
title: XML importer from Danea Easyfatt to Cubecart
date: 2007-07-30 14:56
author: Giacomo Graziosi
comments: true
categories: [php, ]
tags: [cubecart, danea, danea easyfatt, easyfatt, gpl, open source, php, plugin, programming, source]
---
This is ugly, bugged, wrote in PHP (I’m not a PHP programmer) and will probably send death threats emails to all of your customers (really, I didn’t have the time to test it).
If you still want it, <a href="../files/2007/07/cube_importer-01.zip">click to download</a> or just read it online:
{% highlight php %}
<?php
/**
 * ----------------------------------------------------------------------------
 * 
 *  Copyright (C) 2007 Giacomo Graziosi (g.graziosi@gmail.com)
 * 
 * ----------------------------------------------------------------------------
 * 
 *  This program is free software; you can redistribute it and/or modify
 *  it under the terms of the GNU General Public License as published by
 *  the Free Software Foundation; either version 3 of the License, or
 *  (at your option) any later version.
 *
 *  This program is distributed in the hope that it will be useful,
 *  but WITHOUT ANY WARRANTY; without even the implied warranty of
 *  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 *  GNU General Public License for more details.
 *
 *  You should have received a copy of the GNU General Public License
 *  along with this program.  If not, see .
 *
 * ----------------------------------------------------------------------------
**/
 
 
error_reporting(E_ALL);
 
//Ugly importer interface abstraction
class Outputter
{
    private $file_handle;
 
    public function __construct($file)
    {
        $this->file_handle = fopen($file, 'a');
    }
 
    public function __destruct()
    {
        fclose($this->file_handle);
    }
 
    public function write($buffer)
    {
        fwrite($this->file_handle, $buffer."\n##################################\n");
    }
}
 
//Importer implementation for the Cubecart
abstract class Importer
{
    protected $logger;
    protected $dbh;
    protected $xml_products;
 
    public function __construct($import_str, $host, $dbname, $user, $pass, $op)
    {
        $this->logger = $op;
        $this->dbh = new PDO('mysql:host='.$host.';dbname='.$dbname, $user, $pass);
        //$xml = simplexml_load_file($xml_file);
        $this->import_data($import_str);
        $this->update_and_delete();
        $this->insert_new();
    }
 
    public function __destruct()
    {
        //print_r($this->cats);
    }
 
    protected function query_has_row($sql_query, $str_id = "cat_id")
    {
        //echo "$sql_query\n";
        //$s = $this->dbh->query($sql_query);
        //if ($row = $s->fetch())
        foreach ($this->dbh->query($sql_query) as $row)
            return $row[$str_id];
        return false;
    }
 
    abstract protected function import_data($import_str);
    abstract protected function update_and_delete();
    abstract protected function insert_new();
}
 
 
class CubeImporter extends Importer
{
    private $cats = array();
 
    protected function import_data($import_str)
    {
        $xml = simplexml_load_string($import_str);
        $this->xml_products = $xml->Products->Product;
    }
 
    protected function update_and_delete()
    {
        foreach ($this->dbh->query('SELECT * from CubeCart_inventory') as $row)
        {
            foreach ($this->xml_products as $product)
            {
                if( $row['productId'] == $product->InternalID )
                {
                    //print($row['productId']." update\n");
                    $this->update_row($product);
                    continue 2;
                }
            }
            //print($row['productId']." delete\n");
            $this->delete_row($row['productId']);
        }
    }
 
    protected function insert_new()
    {
        foreach ($this->xml_products as $product)
        {
            foreach ($this->dbh->query('SELECT * from CubeCart_inventory') as $row)
            {
                if( $row['productId'] == $product->InternalID )
                {
                    continue 2;
                }
            }
 
            $this->add_row($product);
        }
    }
 
 
 
 
    private function add_cat($cat_name, $cat_fat_id = "0")
    {
        $this->dbh->exec("INSERT INTO CubeCart_category (cat_name, cat_father_id)"
        ."values ('$cat_name', $cat_fat_id)");
 
        //echo "ho inserito ".$this->dbh->lastInsertId()."\n";
        return $this->dbh->lastInsertId();
    }
 
    private function check_categories($xml_product) //needs some refactoring
    {
        if (array_key_exists("Subcategory", $xml_product))
        {
            $a = array_search($xml_product->Subcategory, $this->cats);
            if ($a == false)
            {
                $a = array_search($xml_product->Category, $this->cats);
                if ($a == false)
                {
                    $a = $this->query_has_row("SELECT * FROM CubeCart_category WHERE cat_name = '$xml_product->Category'");
                    if ($a == false)
                    {
                        //inserire cat
                        $a = $this->add_cat($xml_product->Category);
                    }
                    //$this->cats["$xml_product->Category"] = $a;
                }
                //$a deve contenere id di cat
                $fa = $a;
                $a = $this->query_has_row("SELECT * FROM CubeCart_category"
                ." WHERE cat_name = '$xml_product->Subcategory' AND cat_father_id = $fa");
                if ($a == false)
                {
                    $a = $this->add_cat($xml_product->Subcategory, $fa);
                }
                //$this->cats["$xml_product->Subcategory"] = $a;
 
            }
        } else { //If Subcategory doesn't exist then there must be at least a Category
            $a = array_search($xml_product->Category, $this->cats);
            if ($a == false)
            {
                $a = $this->query_has_row("SELECT * FROM CubeCart_category WHERE cat_name = '$xml_product->Category'");
                if ($a == false)
                {
                    $a = $this->add_cat($xml_product->Category);
                }
                //$this->cats["$xml_product->Category"] = $a;
            }
        }
        return $a;
    }
 
 
    private function add_row($xml_product)
    {
        $cat_id = $this->check_categories($xml_product);
        $tax_id = $this->query_has_row("SELECT id FROM CubeCart_taxes WHERE taxName = "."'IVA'", "id");        
        $s = $this->dbh->prepare("INSERT INTO CubeCart_inventory (productID, productCode,"
        ."price, name, cat_id, sale_price, stock_level, taxType)"
        ." VALUES (:productID, :productCode, :price, :name, :cat_id, "
        .":sale_price, :stock_level, :taxType)");
 
        $s->bindParam(':productID', $xml_product->InternalID);
        $s->bindParam(':productCode', $xml_product->Code);
        $s->bindParam(':price', $xml_product->GrossPrice3);
        $s->bindParam(':name', $xml_product->Description);
        $s->bindParam(':cat_id', $cat_id);
        $s->bindParam(':sale_price', $xml_product->GrossPrice3);
        $s->bindParam(':stock_level', $xml_product->AvailableQty);
        $s->bindParam(':taxType', $tax_id);
        $s->execute();
 
 
        $this->dbh->exec("INSERT INTO CubeCart_cats_idx (cat_id, productId)"
        ." VALUES ($cat_id, $xml_product->InternalID)");
        $this->dbh->exec("UPDATE CubeCart_category SET noProducts = noProducts + 1 "
        ."WHERE cat_id = $cat_id");
    }
 
    private function update_row($xml_product)
    {
        $cat_id = $this->check_categories($xml_product);
        $tax_id = $this->query_has_row("SELECT id FROM CubeCart_taxes WHERE taxName = "."'IVA'", "id");    
        $sql = "UPDATE CubeCart_inventory SET productCode = :productCode, price = :price, name = :name, cat_id = :cat_id, "
        ."sale_price = :sale_price, stock_level = :stock_level, taxType = :taxType WHERE productID = :productID";
        $s = $this->dbh->prepare($sql);
 
        $s->bindParam(':productID', $xml_product->InternalID);
        $s->bindParam(':productCode', $xml_product->Code);
        $s->bindParam(':price', $xml_product->GrossPrice3);
        $s->bindParam(':name', $xml_product->Description);
        $s->bindParam(':cat_id', $cat_id);
        $s->bindParam(':sale_price', $xml_product->GrossPrice3);
        $s->bindParam(':stock_level', $xml_product->AvailableQty);
        $s->bindParam(':taxType', $tax_id);
        $this->logger->write($sql);
        $s->execute();
    }
 
    private function delete_row($id)
    {
        //echo("$xml_product->Description\n");
        $cat_id = $this->query_has_row("SELECT cat_id FROM CubeCart_inventory WHERE productId = $id", "cat_id");
        $this->dbh->exec("DELETE FROM CubeCart_inventory WHERE productId = $id");
        $this->dbh->exec("UPDATE CubeCart_category SET noProducts = noProducts - 1 "
        ."WHERE productId = $cat_id");
    }
}
 
 
$op = new Outputter("/path/to/log/asd.txt");
 
if (array_key_exists("file", $_FILES)) {
    move_uploaded_file($_FILES['file']['tmp_name'], "file.xml");
    $xmlstr = file_get_contents("file.xml");
    $op->write($xmlstr);
    $ci = new CubeImporter($xmlstr, 'localhost', 'database_name', 'username', 'password', $op);
}
$op->write(var_export($_FILES, true));
$op->write(var_export($_POST, true));
echo("OK");
?>
{% endhighlight %}

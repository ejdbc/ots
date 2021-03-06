# Auto-increment primary key column {#concept_84535_zh .concept}

Auto increment of primary key columns is a new feature launched by Table Store, and is available for PHP SDK V4.0.0 and later.

This feature allows you to specify a primary key column as an auto-increment column. Table Store automatically generates a new value in this column when you write data to a table. The generated value is the largest value in the column under the same partition key. This feature mainly applies to system design scenarios that require auto-increment primary key columns, such as item IDs on e-commerce websites, user IDs of large websites, post IDs in forums, and message IDs in chat tools.

## Features {#section_ybt_13k_2fb .section}

-   Table Store currently supports multiple primary key columns. The first primary key column is the partition key and cannot be set as an auto-increment column.
-   Except for the partition key, any of other primary key columns can be set as an auto-increment column.
-   Auto increment of a primary key column is implemented at the partition key level because the value is increased based on the partition key.
-   Only one primary key column can be set as an auto-increment column in each table.
-   The automatically generated values in an auto-increment column are 64-bit signed long integers.

## Operations { .section}

Auto increment of primary key columns applies to two types of operations: one to create tables and the other to write data.

-   Create a table

    When creating a table, you only need to set PrimaryKeyOption to PrimaryKeyOptionConst::CONST\_PK\_AUTO\_INCR for the auto-increment primary key column.

    Related operation: CreateTable.

    ```language-php
    function createTable($otsClient) 
    {
        $request = [
            'table_meta' => [
                'table_name' => 'table_name',       // Specify the table name.
                'primary_key_schema' => [
                    ['PK_1', PrimaryKeyTypeConst::CONST_STRING],    // The name and type of the first primary key column (partition key) are PK_1 and STRING, respectively.
                    ['PK_2', PrimaryKeyTypeConst::CONST_INTEGER, PrimaryKeyOptionConst::CONST_PK_AUTO_INCR]
                    // The name and type of the second primary key column are PK_2 and INTEGER, respectively. Set this column as an auto-increment primary key column.
                ]
            ],
            'reserved_throughput' => [
                'capacity_unit' => [         // The reserved read/write throughput: 0 read CUs and 0 write CUs.
                    'read' => 0,
                    'write' => 0
                ]
            ],
            'table_options' => [
                'time_to_live' => -1,             // The table never expires.
                'max_versions' => 1,              // Only one version is saved.
                'deviation_cell_version_in_sec' => 86400   // The valid data version offset, in seconds.
            ]
        ];
        $otsClient->createTable($request);
    }
    
    ```

    -   The first primary key column is the partition key and cannot be set as an auto-increment column.
    -   Only INTEGER columns can be set as auto-increment columns.
    -   Only one auto-increment primary key column is allowed in each table.
-   Write data

    When writing data, you only need to set PrimaryKeyType to PrimaryKeyTypeConst::CONST\_PK\_AUTO\_INCR. The value serves as a placeholder.

    Related operations: PutRow, UpdateRow, and BatchWriteRow.

    ```language-php
    function putRow($otsClient)
    {
        $row = [
            'table_name' => 'table_name',
            'primary_key' => [
                ['PK_1', 'Hangzhou'],                      // A list of primary key names and values.
                ['PK_2', null, PrimaryKeyTypeConst::CONST_PK_AUTO_INCR]    // The auto-increment primary key column. You do not need to specify the value because it is automatically generated by Table Store. You only need to enter the placeholder PrimaryKeyTypeConst::CONST_PK_AUTO_INCR.
            ],
            'attribute_columns' => [              // A list of attribute columns.
                ['name', 'John'],                  // The attribute name, attribute value, attribute type, and timestamp. Ignore the parameters that are not specified.
                ['age', 20],
                ['address', 'Alibaba'],
                ['product', 'OTS'],
                ['married', false]
            ],
            'return_content' => [
                'return_type' => ReturnTypeConst::CONST_PK     // Set return_type to ReturnTypeConst::CONST_PK so that the value of the auto-increment primary key column is returned.
            ]
        ];
        $ret = $otsClient->putRow($row);
        print_r($ret);
    
        $primaryKey = $ret['primary_key'];                  // The obtained primary key value can be used by operations such as GetRow, UpdateRow, and DeleteRow.
        return $primaryKey;
    }
    
    
    ```

    -   When writing data, you only need to add a placeholder to the auto-increment primary key column.
    -   To obtain the value of the auto-increment primary key column that is automatically generated after the data is written to Table Store, set return\_type to ReturnTypeConst::CONST\_PK.


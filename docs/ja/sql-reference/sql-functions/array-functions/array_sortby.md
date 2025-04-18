---
displayed_sidebar: docs
---

# array_sortby

配列内の要素を、別の配列またはラムダ式から変換された配列の昇順に並べ替えます。詳細は [Lambda expression](../Lambda_expression.md) を参照してください。この関数は v2.5 からサポートされています。

2 つの配列の要素は、キーと値のペアのような関係にあります。例えば、b = [7,5,6] は a = [3,1,4] のソートキーです。キーと値のペアの関係に基づいて、2 つの配列の要素は以下のように一対一で対応します。

| **Array** | **Element 1** | **Element 2** | **Element 3** |
| --------- | ------------- | ------------- | ------------- |
| a         | 3             | 1             | 4             |
| b         | 7             | 5             | 6             |

配列 `b` が昇順にソートされると、[5,6,7] になります。それに応じて配列 `a` は [1,4,3] になります。

| **Array** | **Element 1** | **Element 2** | **Element 3** |
| --------- | ------------- | ------------- | ------------- |
| a         | 1             | 4             | 3             |
| b         | 5             | 6             | 7             |

ソートに関与する配列の数が 2 つを超える場合、この関数の目的は、複数の配列列（array1, array2, array3 など）の値に基づいて array0 をソートすることです。ソートルールは以下の通りです：

まず、array1 の対応する要素を比較します。
同じ場合は、array2 の対応する要素を比較します。
このようにして、最後の配列列まで続けます。

**注意:**

ソートに参加する配列の要素は、ソート可能な型または JSON 型でなければなりません。
ソートに参加する配列のサイズは、元の配列と一致している必要があります（NULL 値を除く）。

**例の説明**

以下の 4 つの配列が与えられたとします：

```
array0 = [1, 2, 3, 4, 5]
array1 = [6, 5, 5, 5, 4]
array2 = ["d", "b", "a", "b", "4"]
array3 = ["2023-01-01", "2023-01-04", "2023-01-03", "2023-01-05", "2023-01-02"]
```

ソート手順:
1. array1 を比較します：

* array1 のソートされたインデックス順は [4, 2, 3, 1, 0] です。なぜなら 4 < 5 < 5 < 5 < 6 だからです。

2. array1 の要素 [5, 5, 5] に対して、array2 を比較します：

* array2 の [a, b, b] のソート順は [2, 3, 4] です。なぜなら a < b = b だからです。

3. array1 と array2 の要素 [b, b] に対して、array3 を比較します：

* array3 の [2023-01-04, 2023-01-05] のソート順は [2, 4] です。なぜなら 2023-01-04 < 2023-01-05 だからです。

最終的な array0 のソート結果：

```
array0 = [5, 3, 2, 4, 1]
```

## 構文

```Haskell
array_sortby(array0, array1)
array_sortby(<lambda function>, array0 [, array1...])
array_sortby(array0, array1, [array2, array3...])
```

- `array_sortby(array0, array1)`

   `array1` の昇順に従って `array0` をソートします。

- `array_sortby(<lambda function>, array0 [, array1...])`

   ラムダ関数から返された配列に従って `array0` をソートします。

- `array_sortby(array0, array1, [array2, array3...])`

   複数の配列列（array1, array2, array3 など）の値に基づいて array0 を昇順にソートします。ソートルールは、まず array1 の対応する要素を比較し、同じ場合は array2 の対応する要素を比較し、このようにして最後の配列列まで続けます。

## パラメータ

- `array0`: ソートしたい配列。配列、配列表現、または `null` でなければなりません。配列内の要素はソート可能でなければなりません。
- `array1`: `array0` をソートするために使用されるソート配列。配列、配列表現、または `null` でなければなりません。
- `lambda function`: ソート配列を生成するために使用されるラムダ式。
- `array1, [array2, array3...]`：`array0` をソートするために使用されるソート配列。配列、配列表現、または `null` でなければなりません。

## 戻り値

配列を返します。

## 使用上の注意

- この関数は、配列の要素を昇順にのみソートできます。
- `NULL` 値は、返される配列の先頭に配置されます。
- 配列の要素を降順にソートしたい場合は、[reverse](../string-functions/reverse.md) 関数を使用してください。
- ソート配列 (`array1`) が null の場合、`array0` のデータは変更されません。
- 返される配列の要素は、`array0` の要素と同じデータ型を持ちます。null 値の属性も同じです。
- すべての配列は同じ数の要素を持たなければなりません。そうでない場合、エラーが返されます。

## 例

この関数の使用方法を示すために、以下のテーブルを使用します。

```SQL
CREATE TABLE `test_array` (
  `c1` int(11) NULL COMMENT "",
  `c2` ARRAY<int(11)> NULL COMMENT "",
  `c3` ARRAY<int(11)> NULL COMMENT ""
) ENGINE=OLAP
DUPLICATE KEY(`c1`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`c1`)
PROPERTIES (
"replication_num" = "3",
"storage_format" = "DEFAULT",
"enable_persistent_index" = "true",
"compression" = "LZ4"
);

insert into test_array values
(1,[4,3,5],[82,1,4]),
(2,null,[23]),
(3,[4,2],[6,5]),
(4,null,null),
(5,[],[]),
(6,NULL,[]),
(7,[],null),
(8,[null,null],[3,6]),
(9,[432,21,23],[5,4,null]);

select * from test_array order by c1;
+------+-------------+------------+
| c1   | c2          | c3         |
+------+-------------+------------+
|    1 | [4,3,5]     | [82,1,4]   |
|    2 | NULL        | [23]       |
|    3 | [4,2]       | [6,5]      |
|    4 | NULL        | NULL       |
|    5 | []          | []         |
|    6 | NULL        | []         |
|    7 | []          | NULL       |
|    8 | [null,null] | [3,6]      |
|    9 | [432,21,23] | [5,4,null] |
+------+-------------+------------+
9 rows in set (0.00 sec)
```

例 1: `c2` に従って `c3` をソートします。この例では、比較のために array_sort() の結果も提供します。

```Plaintext
select c1, c3, c2, array_sort(c2), array_sortby(c3,c2)
from test_array order by c1;
+------+------------+-------------+----------------+----------------------+
| c1   | c3         | c2          | array_sort(c2) | array_sortby(c3, c2) |
+------+------------+-------------+----------------+----------------------+
|    1 | [82,1,4]   | [4,3,5]     | [3,4,5]        | [1,82,4]             |
|    2 | [23]       | NULL        | NULL           | [23]                 |
|    3 | [6,5]      | [4,2]       | [2,4]          | [5,6]                |
|    4 | NULL       | NULL        | NULL           | NULL                 |
|    5 | []         | []          | []             | []                   |
|    6 | []         | NULL        | NULL           | []                   |
|    7 | NULL       | []          | []             | NULL                 |
|    8 | [3,6]      | [null,null] | [null,null]    | [3,6]                |
|    9 | [5,4,null] | [432,21,23] | [21,23,432]    | [4,null,5]           |
+------+------------+-------------+----------------+----------------------+
```

例 2: ラムダ式から生成された `c2` に基づいて配列 `c3` をソートします。この例は例 1 と同等です。比較のために array_sort() の結果も提供します。

```Plaintext
select
    c1,
    c3,
    c2,
    array_sort(c2) as sorted_c2_asc,
    array_sortby((x,y) -> y, c3, c2) as sorted_c3_by_c2
from test_array order by c1;
+------+------------+-------------+---------------+-----------------+
| c1   | c3         | c2          | sorted_c2_asc | sorted_c3_by_c2 |
+------+------------+-------------+---------------+-----------------+
|    1 | [82,1,4]   | [4,3,5]     | [3,4,5]       | [82,1,4]        |
|    2 | [23]       | NULL        | NULL          | [23]            |
|    3 | [6,5]      | [4,2]       | [2,4]         | [5,6]           |
|    4 | NULL       | NULL        | NULL          | NULL            |
|    5 | []         | []          | []            | []              |
|    6 | []         | NULL        | NULL          | []              |
|    7 | NULL       | []          | []            | NULL            |
|    8 | [3,6]      | [null,null] | [null,null]   | [3,6]           |
|    9 | [5,4,null] | [432,21,23] | [21,23,432]   | [4,null,5]      |
+------+------------+-------------+---------------+-----------------+
```

例 3: `c2+c3` の昇順に基づいて配列 `c3` をソートします。

```Plain
select
    c3,
    c2,
    array_map((x,y)-> x+y,c3,c2) as sum,
    array_sort(array_map((x,y)-> x+y, c3, c2)) as sorted_sum,
    array_sortby((x,y) -> x+y, c3, c2) as sorted_c3_by_sum
from test_array where c1=1;
+----------+---------+----------+------------+------------------+
| c3       | c2      | sum      | sorted_sum | sorted_c3_by_sum |
+----------+---------+----------+------------+------------------+
| [82,1,4] | [4,3,5] | [86,4,9] | [4,9,86]   | [1,4,82]         |
+----------+---------+----------+------------+------------------+
```

```SQL
CREATE TABLE test_array_sortby_muliti (
    id INT(11) not null,
    array_col1 ARRAY<INT>,
    array_col2 ARRAY<DOUBLE>,
    array_col3 ARRAY<VARCHAR(20)>,
    array_col4 ARRAY<DATE>
) ENGINE=OLAP
DUPLICATE KEY(id)
COMMENT "OLAP"
DISTRIBUTED BY HASH(id)
PROPERTIES (
    "replication_num" = "1",
    "storage_format" = "DEFAULT",
    "enable_persistent_index" = "true",
    "compression" = "LZ4"
);

INSERT INTO test_array_sortby_multi VALUES
(1, [4, 3, 5], [1.1, 2.2, 2.2], ['a', 'b', 'c'], ['2023-01-01', '2023-01-02', '2023-01-03']),
(2, [6, 7, 8], [6.6, 5.5, 6.6], ['d', 'e', 'd'], ['2023-01-04', '2023-01-05', '2023-01-06']),
(3, NULL, [7.7, 8.8, 8.8], ['g', 'h', 'h'], ['2023-01-07', '2023-01-08', '2023-01-09']),
(4, [9, 10, 11], NULL, ['k', 'k', 'j'], ['2023-01-10', '2023-01-12', '2023-01-11']),
(5, [12, 13, 14], [10.10, 11.11, 11.11], NULL, ['2023-01-13', '2023-01-14', '2023-01-15']),
(6, [15, 16, 17], [14.14, 13.13, 14.14], ['m', 'o', 'o'], NULL),
(7, [18, 19, 20], [16.16, 16.16, 18.18], ['p', 'p', 'r'], ['2023-01-16', NULL, '2023-01-18']),
(8, [21, 22, 23], [19.19, 20.20, 19.19], ['a', 't', 'a'], ['2023-01-19', '2023-01-20', '2023-01-21']),
(9, [24, 25, 26], NULL, ['y', 'y', 'z'], ['2023-01-25', '2023-01-24', '2023-01-26']),
(10, [24, 25, 26], NULL, ['y', 'y', 'z'], ['2023-01-25', NULL, '2023-01-26']);

select * from test_array_sortby_multi order by id asc;
+------+------------+---------------------+---------------+------------------------------------------+
| id   | array_col1 | array_col2          | array_col3    | array_col4                               |
+------+------------+---------------------+---------------+------------------------------------------+
|    1 | [4,3,5]    | [1.1,2.2,2.2]       | ["a","b","c"] | ["2023-01-01","2023-01-02","2023-01-03"] |
|    2 | [6,7,8]    | [6.6,5.5,6.6]       | ["d","e","d"] | ["2023-01-04","2023-01-05","2023-01-06"] |
|    3 | NULL       | [7.7,8.8,8.8]       | ["g","h","h"] | ["2023-01-07","2023-01-08","2023-01-09"] |
|    4 | [9,10,11]  | NULL                | ["k","k","j"] | ["2023-01-10","2023-01-12","2023-01-11"] |
|    5 | [12,13,14] | [10.1,11.11,11.11]  | NULL          | ["2023-01-13","2023-01-14","2023-01-15"] |
|    6 | [15,16,17] | [14.14,13.13,14.14] | ["m","o","o"] | NULL                                     |
|    7 | [18,19,20] | [16.16,16.16,18.18] | ["p","p","r"] | ["2023-01-16",null,"2023-01-18"]         |
|    8 | [21,22,23] | [19.19,20.2,19.19]  | ["a","t","a"] | ["2023-01-19","2023-01-20","2023-01-21"] |
|    9 | [24,25,26] | NULL                | ["y","y","z"] | ["2023-01-25","2023-01-24","2023-01-26"] |
|   10 | [24,25,26] | NULL                | ["y","y","z"] | ["2023-01-25",null,"2023-01-26"]         |
+------+------------+---------------------+---------------+------------------------------------------+

例 1: `array_col2`, `array_col3` に従って `array_col1` をソートします。

```Plaintext
select id, array_col1, array_col2, array_col3, array_sortby(array_col1, array_col2, array_col3) from test_array_sortby_multi order by id asc;
+------+------------+---------------------+---------------+--------------------------------------------------------+
| id   | array_col1 | array_col2          | array_col3    | array_sortby(array_col1, array_col2, array_col3)       |
+------+------------+---------------------+---------------+--------------------------------------------------------+
|    1 | [4,3,5]    | [1.1,2.2,2.2]       | ["a","b","c"] | [4,3,5]                                                |
|    2 | [6,7,8]    | [6.6,5.5,6.6]       | ["d","e","d"] | [7,6,8]                                                |
|    3 | NULL       | [7.7,8.8,8.8]       | ["g","h","h"] | NULL                                                   |
|    4 | [9,10,11]  | NULL                | ["k","k","j"] | [11,9,10]                                              |
|    5 | [12,13,14] | [10.1,11.11,11.11]  | NULL          | [12,13,14]                                             |
|    6 | [15,16,17] | [14.14,13.13,14.14] | ["m","o","o"] | [16,15,17]                                             |
|    7 | [18,19,20] | [16.16,16.16,18.18] | ["p","p","r"] | [18,19,20]                                             |
|    8 | [21,22,23] | [19.19,20.2,19.19]  | ["a","t","a"] | [21,23,22]                                             |
|    9 | [24,25,26] | NULL                | ["y","y","z"] | [24,25,26]                                             |
|   10 | [24,25,26] | NULL                | ["y","y","z"] | [24,25,26]                                             |
+------+------------+---------------------+---------------+--------------------------------------------------------+
```

例 2: `array_col2`, `array_col3`, `array_col4` に従って `array_col1` をソートします。

```Plaintext
select id, array_col1, array_col2, array_col3, array_col4, array_sortby(array_col1, array_col2, array_col3, array_col4) from test_array_sortby_multi order by id asc;
+------+------------+---------------------+---------------+------------------------------------------+--------------------------------------------------------------------+
| id   | array_col1 | array_col2          | array_col3    | array_col4                               | array_sortby(array_col1, array_col2, array_col3, array_col4)       |
+------+------------+---------------------+---------------+------------------------------------------+--------------------------------------------------------------------+
|    1 | [4,3,5]    | [1.1,2.2,2.2]       | ["a","b","c"] | ["2023-01-01","2023-01-02","2023-01-03"] | [4,3,5]                                                            |
|    2 | [6,7,8]    | [6.6,5.5,6.6]       | ["d","e","d"] | ["2023-01-04","2023-01-05","2023-01-06"] | [7,6,8]                                                            |
|    3 | NULL       | [7.7,8.8,8.8]       | ["g","h","h"] | ["2023-01-07","2023-01-08","2023-01-09"] | NULL                                                               |
|    4 | [9,10,11]  | NULL                | ["k","k","j"] | ["2023-01-10","2023-01-12","2023-01-11"] | [11,9,10]                                                          |
|    5 | [12,13,14] | [10.1,11.11,11.11]  | NULL          | ["2023-01-13","2023-01-14","2023-01-15"] | [12,13,14]                                                         |
|    6 | [15,16,17] | [14.14,13.13,14.14] | ["m","o","o"] | NULL                                     | [16,15,17]                                                         |
|    7 | [18,19,20] | [16.16,16.16,18.18] | ["p","p","r"] | ["2023-01-16",null,"2023-01-18"]         | [19,18,20]                                                         |
|    8 | [21,22,23] | [19.19,20.2,19.19]  | ["a","t","a"] | ["2023-01-19","2023-01-20","2023-01-21"] | [21,23,22]                                                         |
|    9 | [24,25,26] | NULL                | ["y","y","z"] | ["2023-01-25","2023-01-24","2023-01-26"] | [25,24,26]                                                         |
|   10 | [24,25,26] | NULL                | ["y","y","z"] | ["2023-01-25",null,"2023-01-26"]         | [25,24,26]                                                         |
+------+------------+---------------------+---------------+------------------------------------------+--------------------------------------------------------------------+
```

## 参考文献

[array_sort](array_sort.md)
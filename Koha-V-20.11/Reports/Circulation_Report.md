# SQL Reports - Circulation
### 1)Complete Accession Register With Item Type Filter
```MySQL
SELECT  items.barcode,items.dateaccessioned,biblio.title,biblio.author,items.price,items.replacementprice,
items.itemcallnumber,items.ccode,items.itype,biblioitems.volume,biblioitems.isbn,biblioitems.issn,
biblio.copyrightdate AS 'Publishing Year',biblioitems.publishercode,biblioitems.pages,
ExtractValue(biblio_metadata.metadata, '//datafield[@tag="650"]/subfield[@code="a"]') AS Subject,
ExtractValue(biblio_metadata.metadata, '//datafield[@tag="654"]/subfield[@code="a"]') AS Keywords
FROM biblio_metadata
LEFT JOIN biblioitems on (biblio_metadata.biblionumber=biblioitems.biblionumber)
LEFT JOIN items on (items.biblioitemnumber=biblioitems.biblioitemnumber)  
LEFT JOIN biblio on (biblio_metadata.biblionumber=biblio.biblionumber) 
WHERE items.itype=<<itype|itemtypes>>
ORDER BY items.barcode ASC
```
### 2) Complete Accession Register With Item Status
> Status `Withdrawn` , `Checked_out` , `Lost` , `Damaged` , `Available` 
```MySQL
SELECT items.barcode,
biblio.title,
EXTRACTVALUE( b.metadata, '//datafield[@tag="654"]/subfield[@code>="a"]' ) AS Keyword,
items.dateaccessioned,items.itemcallnumber,EXTRACTVALUE( b.metadata, '//datafield[@tag="952"]/subfield[@code>="o"]' ) AS ClassificationNo,
biblio.author,biblioitems.editionstatement,
biblioitems.volume,biblioitems.place,biblioitems.publishercode,biblio.copyrightdate,items.booksellerid,
items.stocknumber AS inventory_details,
items.price AS Purchase_price,EXTRACTVALUE( b.metadata, '//datafield[@tag="952"]/subfield[@code>="g"]' ) AS Actual_Price,
items.ccode As Collection_code,
CASE
when items.onloan IS NOT NULL THEN 'Checked_out'
WHEN items.withdrawn !=0 THEN 'Withdrawn'
WHEN items.itemlost !=0 THEN 'Lost'
WHEN items.damaged !=0  THEN 'Damaged'
ELSE 'Available'
END
AS Status
from items
left join biblio on items.biblionumber = biblio.biblionumber
left join biblioitems on biblio.biblionumber=biblioitems.biblionumber
left join  biblio_metadata b on biblioitems.biblionumber = b.biblionumber
order by cast(substr(items.barcode,3) as int) ASC
```
### 3) Missing Barcodes - For numeric barcodes only

```MySQL
SELECT 
    concat('From : ',(CAST(i1.barcode AS int) + 1),' To : ',
        (SELECT MIN(CAST(i3.barcode AS int)) -1 FROM items i3 WHERE CAST(i3.barcode AS int) > CAST(i1.barcode AS int))
    ) AS "Missing Barcodes From : To"
FROM items i1
WHERE 
	CAST(i1.barcode AS int) BETWEEN <<Enter from barcode>> AND <<Enter to barcode>> 
    AND NOT EXISTS (SELECT CAST(i2.barcode AS int) FROM items i2 WHERE CAST(i2.barcode AS int) = CAST(i1.barcode AS int) + 1)
HAVING "Missing Barcodes from : To" IS NOT NULL
```
### 4) Missing Barcodes - For Alphanumeric Barcodes Only
> This report only gives results if the barcode starts with a single alphabet ex . Barcode B0023, B0024<br>
> If your barcode starts with two or more alphabets, update the report to suit your needs. All you have to do is update the value in all `SUBSTR` functions
```MySQL
SELECT 
    SUBSTR(i1.barcode,1,1) as "barcode prifix",(CAST(SUBSTR(i1.barcode,2) AS int) + 1) AS gap_starts_at, 
    (SELECT MIN(CAST(SUBSTR(i3.barcode,2) AS int)) -1 FROM items i3 
        WHERE SUBSTR(i3.barcode,1,1) = <<Enter prefix>> AND 
        CAST(SUBSTR(i3.barcode,2) AS int) > CAST(SUBSTR(i1.barcode,2) AS int)) AS gap_ends_at
FROM items i1
WHERE 
	SUBSTR(i1.barcode,1,1) = <<Enter prefix>> 
    AND
	CAST(SUBSTR(i1.barcode,2) AS int) BETWEEN <<Enter from barcode>> AND <<Enter to barcode>> 
AND NOT EXISTS (SELECT CAST(SUBSTR(i2.barcode,2) AS int) FROM items i2 
                WHERE SUBSTR(i2.barcode,1,1) = <<Enter prefix>>  and
                CAST(SUBSTR(i2.barcode,2) AS int) = CAST(SUBSTR(i1.barcode,2) AS int) + 1)
HAVING gap_ends_at IS NOT NULL
```

### 5)
```MySQL

```
### 6)
```MySQL

```
### 7)
```MySQL

```
### 8)
```MySQL

```
### 9)
```MySQL

```
### 10)
```MySQL

```
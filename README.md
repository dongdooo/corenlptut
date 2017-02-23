# การใช้ CoreNLP เบื้องต้น

## โปรแกรมที่ต้องมีก่อน

โปรแกรมที่จำเป็นต้องใช้ก่อนติดตั้ง CoreNLP

1. Java VM
2. curl
3. BASH
4. find

## การติดตั้ง

1. ดาวโหลด CoreNLP http://nlp.stanford.edu/software/stanford-corenlp-full-2016-10-31.zip
2. unzip stanford-corenlp-full-2016-10-31.zip


## เริ่มให้ server ทำงาน

1. cd stanford-corenlp-full-2016-10-31/
2. สั่ง

      java -mx4g -cp "*" edu.stanford.nlp.pipeline.StanfordCoreNLPServer -port 9000 -timeout 15000

3. เปิดหน้าต่างที่รัน server ทิ้งไว้ โดยจะเห็นข้อความทำนองนี้
````
$ java -mx4g -cp "*" edu.stanford.nlp.pipeline.StanfordCoreNLPServer -port 9000 -timeout 15000
[main] INFO CoreNLP - --- StanfordCoreNLPServer#main() called ---
[main] INFO CoreNLP - setting default constituency parser
[main] INFO CoreNLP - warning: cannot find edu/stanford/nlp/models/srparser/englishSR.ser.gz
[main] INFO CoreNLP - using: edu/stanford/nlp/models/lexparser/englishPCFG.ser.gz instead
[main] INFO CoreNLP - to use shift reduce parser download English models jar from:
[main] INFO CoreNLP - http://stanfordnlp.github.io/CoreNLP/download.html
[main] INFO CoreNLP -     Threads: 4
[main] INFO CoreNLP - Starting server...
[main] INFO CoreNLP - StanfordCoreNLPServer listening at /0:0:0:0:0:0:0:0:
````

## ใช้งาน CoreNLP

1. ในอีกหน้าต่างหนึ่ง สั่งคำสั่งนี้

````bash
curl --data 'The quick brown fox jumped over the lazy dog.' 'http://localhost:9000/?properties={%22annotators%22%3A%22tokenize%2Cssplit%2Cpos%22%2C%22outputFormat%22%3A%22json%22}' -o -
````

จะได้ผลลัพธ์ตามด้านล่าง

````
$ curl --data 'The quick brown fox jumped over the lazy dog.' 'http://localhost:9000/?properties={%22annotators%22%3A%22tokenize%2Cssplit%2Cpos%22%2C%22outputFormat%22%3A%22json%22}' -o -

{"sentences":[{"index":0,"tokens":[{"index":1,"word":"The","originalText":
"The","characterOffsetBegin":0,"characterOffsetEnd":3,"pos":"DT",
"before":"","after":" "},{"index":2,"word":"quick","originalText":"quick",
"characterOffsetBegin":4,"characterOffsetEnd":9,"pos":"JJ",
"before":" ","after":" "},{"index":3,"word":"brown","originalText":
"brown","characterOffsetBegin":10,"characterOffsetEnd":15,"pos":"JJ",
"before":" ","after":" "},{"index":4,"word":"fox","originalText":"fox",
"characterOffsetBegin":16,"characterOffsetEnd":19,"pos":"NN",
"before":" ","after":" "},{"index":5,"word":"jumped","originalText":
"jumped","characterOffsetBegin":20,"characterOffsetEnd":26,"pos":"VBD",
"before":" ","after":" "},{"index":6,"word":"over","originalText":
"over","characterOffsetBegin":27,"characterOffsetEnd":31,"pos":
"IN","before":" ","after":" "},{"index":7,"word":"the","originalText":
"the","characterOffsetBegin":32,"characterOffsetEnd":35,"pos":"DT",
"before":" ","after":" "},{"index":8,"word":"lazy","originalText":
"lazy","characterOffsetBegin":36,"characterOffsetEnd":40,"pos":
"JJ","before":" ","after":" "},{"index":9,"word":"dog","originalText":
"dog","characterOffsetBegin":41,"characterOffsetEnd":44,"pos":"NN",
"before":" ","after":""},{"index":10,"word":".","originalText":".",
"characterOffsetBegin":44,"characterOffsetEnd":45,"pos":".",
"before":"","after":""}]}]}$ 
$ 
````

## กรณีที่ corpus มีหลายไฟล์

1. ดาวน์โหลดไฟล์ tagpos.sh จาก repository นี้
2. สั่ง chmod 755 tagpos.sh
3. ใช้คำสั่งนี้ สมมุติว่าไฟล์ทั้งหมดอยู่ใน folder ชื่อ corpus และชื่อไฟล์และชื่อ folder ไม่มีช่องว่าง ไฟล์ทุกไฟล์มีส่วนต่อท้ายคือ .txt (หมายเหตุ: ใน repository นี้ corpus เล็ก ๆ มาก ๆ สามารถ close ไปเป็นตัวอย่าง)

````bash
find corpus -name '*.txt' | xargs -I {} ./tagpos.sh '{}'
````

พอรันคำสั่งข้างบนมาแล้วจะได้ผลลัพธ์เป็นไฟล์ .txt.json 

ถ้าหากใช้ corpus ตัวอย่างใน folder ก็จะมีไฟล์แบบข้างล่าง

````
corpus/b1.txt.json
corpus/a1.txt.json
corpus/a1.txt
corpus/b1.txt
````

ไฟล์ .txt.json สามารถนำไปประมวลผลต่อได้เลย

## กรณีที่ไฟล์ใหญ่

ผมเพิ่ม tagpos_mline.sh ขึ้นมาครับแทนที่จะส่งข้อความไปทีเดียว มันจะตัดบรรทัดก่อน แล้วค่อยส่งไปให้ CoreNLP ไปวิเคราะห์ที่ละบรรทัดครับ

ผลลัพธ์ที่ได้ก็จะเปลี่ยนไปจาก tagpos.sh ด้วยครับ จะกลายเป็นว่ามีผลลัพธ์จากการวิเคราะห์หลายบรรทัด เวลาใช้ตัวอ่าน JSON ก็ต้องตัดบรรทัดก่อนแล้วถึงเอาไปเข้าตัวอ่าน JSON ครับ

## การแจงประโยค

### แจงประโยคโดยเรียกจาก Shell script

````bash
./parse.sh <ชื่อไฟล์>
````

### แจงประโยคผ่าน Curl

````bash
curl --data 'The quick brown fox jumped over the lazy dog.' 'http://localhost:9000/?properties={%22annotators%22%3A%22tokenize%2Cssplit%2Cpos%2Cparse%22%2C%22outputFormat%22%3A%22json%22}' -o -
````

### ผลการแจงประโยค

ผลการแจงประโยคจะออกมาทำนองนี้

````json
{"sentences":[
  {"index":0,
   "parse":"(ROOT\n  (S\n    
                        (NP (PRP I))\n    
                        (VP (VBP am)\n      
                            (NP (DT a) (NN man)))\n    (. .)))",
   "basic-dependencies":[
        {"dep":"ROOT",
         "governor":0,
         "governorGloss":"ROOT",
         "dependent":4,
         "dependentGloss":"man"}
...
````

## Python

### ตัวอย่างการใช้ JSON Parser      

````
>>> import json
>>> x = json.loads('{"A":1,"B":2}')
>>> x["A"]
1
>>> 
````

#### ตัวอย่างการใช้ HTTP Requests

````
>>> import requests
>>> r = requests.post('http://localhost:9000?properties={%22annotators%22%3A%22tokenize%2Cssplit%2Cpos%2Cparse%22%2C%22outputFormat%22%3A%22json%22}', data="This is a book.")
>>> r.json()
{'sentences': [{'basic-dependencies': [{'governor': 0, 'dependent': 4, 'dependentGloss': 'book', 'governorGloss': 'ROOT', 'dep': 'ROOT'}, {'governor': 4, 'dependent': 1, 'dependentGloss': 'This', 'governorGloss': 'book', 'dep': 'nsubj'}, {'governor': 4, 'dependent': 2, 'dependentGloss': 'is', 'governorGloss': 'book', 'dep': 'cop'}, {'governor': 4, 'dependent': 3, 'dependentGloss': 'a', 'governorGloss': 'book', 'dep': 'det'}, {'governor': 4, 'dependent': 5, 'dependentGloss': '.', 'governorGloss': 'book', 'dep': 'punct'}], 'parse': '(ROOT\n  (S\n    (NP (DT This))\n    (VP (VBZ is)\n      (NP (DT a) (NN book)))\n    (. .)))', 'tokens': [{'characterOffsetBegin': 0, 'before': '', 'word': 'This', 'index': 1, 'characterOffsetEnd': 4, 'after': ' ', 'pos': 'DT', 'originalText': 'This'}, {'characterOffsetBegin': 5, 'before': ' ', 'word': 'is', 'index': 2, 'characterOffsetEnd': 7, 'after': ' ', 'pos': 'VBZ', 'originalText': 'is'}, {'characterOffsetBegin': 8, 'before': ' ', 'word': 'a', 'index': 3, 'characterOffsetEnd': 9, 'after': ' ', 'pos': 'DT', 'originalText': 'a'}, {'characterOffsetBegin': 10, 'before': ' ', 'word': 'book', 'index': 4, 'characterOffsetEnd': 14, 'after': '', 'pos': 'NN', 'originalText': 'book'}, {'characterOffsetBegin': 14, 'before': '', 'word': '.', 'index': 5, 'characterOffsetEnd': 15, 'after': '', 'pos': '.', 'originalText': '.'}], 'collapsed-dependencies': [{'governor': 0, 'dependent': 4, 'dependentGloss': 'book', 'governorGloss': 'ROOT', 'dep': 'ROOT'}, {'governor': 4, 'dependent': 1, 'dependentGloss': 'This', 'governorGloss': 'book', 'dep': 'nsubj'}, {'governor': 4, 'dependent': 2, 'dependentGloss': 'is', 'governorGloss': 'book', 'dep': 'cop'}, {'governor': 4, 'dependent': 3, 'dependentGloss': 'a', 'governorGloss': 'book', 'dep': 'det'}, {'governor': 4, 'dependent': 5, 'dependentGloss': '.', 'governorGloss': 'book', 'dep': 'punct'}], 'index': 0, 'collapsed-ccprocessed-dependencies': [{'governor': 0, 'dependent': 4, 'dependentGloss': 'book', 'governorGloss': 'ROOT', 'dep': 'ROOT'}, {'governor': 4, 'dependent': 1, 'dependentGloss': 'This', 'governorGloss': 'book', 'dep': 'nsubj'}, {'governor': 4, 'dependent': 2, 'dependentGloss': 'is', 'governorGloss': 'book', 'dep': 'cop'}, {'governor': 4, 'dependent': 3, 'dependentGloss': 'a', 'governorGloss': 'book', 'dep': 'det'}, {'governor': 4, 'dependent': 5, 'dependentGloss': '.', 'governorGloss': 'book', 'dep': 'punct'}]}]}
>>> 
````

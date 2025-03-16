![[Pasted image 20250226111535.png]]
![[Pasted image 20250226111634.png]]
![[Pasted image 20250226111706.png]]
![[Pasted image 20250226111726.png]]
![[Pasted image 20250226111755.png]]
![[Pasted image 20250226111844.png]]
![[Pasted image 20250226111950.png]]
![[Pasted image 20250226112144.png]]
![[Pasted image 20250226112545.png]]
![[Pasted image 20250226112603.png]]
![[Pasted image 20250226112650.png]]
![[Pasted image 20250226112831.png]]
![[Pasted image 20250226112902.png]]
![[Pasted image 20250226112944.png]]
![[Pasted image 20250226113258.png]]
![[Pasted image 20250226113352.png]]
![[Pasted image 20250226113452.png]]
![[Pasted image 20250226113729.png]]
![[Pasted image 20250226113922.png]]
![[Pasted image 20250226114015.png]]
![[Pasted image 20250226114352.png]]

![[Pasted image 20250226183416.png]]
![[Pasted image 20250226183536.png]]
![[Pasted image 20250226183640.png]]
![[Pasted image 20250226183712.png]]

![[Pasted image 20250226183733.png]]

```
# создать индекс с определенным числом шардов
PUT transformators
{
	
	"settings": {
	
	"number_of_shards": 5,
	
	"number_of_replicas": "2"
	
	}

}

# вставить документ
POST transformators/_doc
{

	"Трехфазные двухобмоточные трансформаторы 330 кВ; ТРДНС-40000/330 ": {
	
	"Тип": "ТРДНС-40000/330 ",
	
	"Sном, МВА": 40,
	
	"Пределы регулирования ВН НН": "±8*1,5% ",
	
	"ВН": 330,
	
	"НН": "6,3/6,3;6,3/10,5;10,5/10,5",
	
	"Uк,%": 11.0,
	
	" ΔРк, кВт": 180,
	
	"Рх, кВт": 80,
	
	"Iх, %": 1.4,
	
	"Rт, Ом": 12.3,
	
	"Хт, Ом": 299.0,
	
	"ΔQх, кВАр": 560
	
	},

	"Трехфазные двухобмоточные трансформаторы 330 кВ; ТРДЦН-63000/330 ": {
	
	"Тип": "ТРДЦН-63000/330 ",
	
	"Sном, МВА": 63,
	
	"Пределы регулирования ВН НН": "±1,5% ",
	
	"ВН": 330,
	
	"НН": "6,3/6,3;6,3/10,5;10,5/10,5",
	
	"Uк,%": 11.0,
	
	" ΔРк, кВт": 265,
	
	"Рх, кВт": 120,
	
	"Iх, %": 0.7,
	
	"Rт, Ом": 7.3,
	
	"Хт, Ом": 190.0,
	
	"ΔQх, кВАр": 441

	}
}

# Получить айтем по id
GET my_shini_index/_doc/uXmHQpUBTgYQC1Bz2NYy

# Добавить новое поле
POST my_shini_index/_update/uXmHQpUBTgYQC1Bz2NYy

{

	"doc": {
	
	"Conductor resistance": "3 kOm"
	
	}

}
# Узнать на какой shard улетел документ
GET my_shini_index/_search_shards?routing=uXmHQpUBTgYQC1Bz2NYy

# Инфо про индекс - документы в индексе
GET my_map_index/_search
```

![[Pasted image 20250227094045.png]]
```
# По дефолту применяется стандартный анализатор
POST /_analyze
{"text": "Мама мыла раму", "analyzer": "standard"}


#  Создать индекс с кастомным 
PUT my_map_index_2
{
	"mappings": {	
		"properties": {\	
			"name": {			
			"type": "text",
			"fields": {
				"keyword": {
					"type": "keyword",
					"ignore_above": 256
				}
			
			}
	
		},
	
		"surname": {
		
			"type": "text",
			
			"fields": {
			
				"keyword": {
				
				"type": "keyword",
				
				"ignore_above": 256
			
				}
		
			}
		
		},
	
	"rating": {"type": "integer"}
	
	}
	
	}

}
```

![[Pasted image 20250227094445.png]]
![[Pasted image 20250227094718.png]]
![[Pasted image 20250227094820.png]]
![[Pasted image 20250227095052.png]]
![[Pasted image 20250227145822.png]]
![[Pasted image 20250227145910.png]]
![[Pasted image 20250227145948.png]]
![[Pasted image 20250227150028.png]]
![[Pasted image 20250227151814.png]]
![[Pasted image 20250227153023.png]]
![[Pasted image 20250227153059.png]]
![[Pasted image 20250227153541.png]]
![[Pasted image 20250227153624.png]]
![[Pasted image 20250227164454.png]]
![[Pasted image 20250227170319.png]]
![[Pasted image 20250227171201.png]]
![[Pasted image 20250227171235.png]]
![[Pasted image 20250227171313.png]]
![[Pasted image 20250227171339.png]]

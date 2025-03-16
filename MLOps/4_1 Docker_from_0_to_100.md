
![[Pasted image 20250102174145.png]]

![[Pasted image 20250102174204.png]]

![[Pasted image 20250102174507.png]]
![[Pasted image 20250102174928.png]]

1. `docker pull 'image name'` - скачать image
2. `docker run 'image name'`- запустить контейнер
3. `docker rm 'container id'` - удалить контейнер
4. `docker images` - список из images
5. `docker rmi 'image id'`- удалить image
6. `docker run ubuntu sleep 5`- запустить контейнер и ждать 5 сек
7. `docker run -d ubuntu sleep 5`- запустить контейнер в detach режиме и ждать 5 сек
8. `docker ps` - список работающих контейнеров
9. `docker ps -a` - список запущенных контейнеров
10. `docker start 'container id'` - перезапустить контейнер
11. `docker pause 'container_id'` - остановить контейнер
12. `docker unpause 'container_id'` - убрать паузу
13. `docker stop 'container_id'` - остановить контейнер (может занят много времени)
14. `docker kill 'container_id'` - остановить контейнер
15. `docker run --rm -d ubuntu sleep 900` - удалить контейнер после остановки автоматически
16. `docker inspect 'container id'`
17. `docker stats container id` - сколько памяти используется, нагрузка на сеть и тд
18. `docker run -d  --name MyContainer ubuntu echo 'output message'` - запустить контейнер с определенным именем
19. `docker logs 'container_id'` - логи контейнера
20. `docker exec -it container_name /bin/bash` - зайти внутрь контейнера
21. `docker system prune -a --volume` - удалить все отстановленные контейнера и images
![[Pasted image 20250102200644.png]]
22. `docker run -p 8080:80 nginx` - запросы потупающие на порт 8080 сервера перенаправляются в порт 80 контейнера
23. `netstat tulpen` - показывает список открытых портов
24. `docker run --name DB-mysql -e MYSQL_ROOT_PASSWORD=root -d mysql` - запуск с переменными окружения
25. `env` - переменные окружения
26. `export var=1` - установить переменную окружения
27. ![[Pasted image 20250104112041.png]]
28. ![[Pasted image 20250104112321.png]]
29. `docker volume ls` - список volumes
30. `docker volume rm 'volume_name'` - удалить volume
31. `docker volume create 'volume_name'` - создать volume
32. ![[Pasted image 20250104114937.png]]
33. ![[Pasted image 20250104115341.png]]
34. `docker network ls` - список сетей в докере
35. `docker network create 'network_name'` - создать сеть с названием network_name
36. `docker network create -d bridge --subnet 192.168.10.0/24 --gateway 192.168.10.1 'network_name'` - создать сеть с названием network_name и определенными адресами
37. `docker network rm 'network_name'` - удалить сеть
38. Если мы хотим чтобы контейнера могли обмениватся по названию (пинг по названию) надо сначала создать одну сеть, а потом внутри нее создать контейнера
39. `docker network connect 'net_name' 'container_name'` - связать контейнер с сетью
40. `docker network disconnect 'net_id' 'container_name'` - отвязать контейнер от сети
41. ![[Pasted image 20250106221129.png]]
42. `docker build .` - запуск в той директории, где находится файл Dockerfile
43. `docker tag image_id 'new_name'` - переименовать image
44. `docker image inspect mydocker:v01` - посмотреть какие команды запускаются
45. `docker build -t 'image_name' .` - запуск в той директории, где находится файл Dockerfile, присвоить имя image_name
46. `docker run -it --rm --name mydocker myimage:v01 /bin/bash` - запустить докер с названием mydocker (image - myimage:v01) в интерактивном режиме
47. `ENTRYPOINT` - изменяемая команда. Если прописать  ENTRYPOINT - мы не сможем зайти в интерактивный режим, при запуске контейнера просто будет принт и все. То что стоит после ENTRYPOINT нельзя изменить
48. `CMD` - неизменяемая команда. Например если в докер файле прописано `CMD ["echo", "Hello my first docker"]`  запустить докер с другим текстом, то принтится будет это сообщение
49.  В докер файле вместо `CMD echo "Hello my first docker"` лучше написать `CMD ['echo", "Hello my first docker"]`
50.   `EXPOSE 80` - 80 порт будет открыт у контейнера
51.  `docker run -d --rm --name myname -P image_name` - все открытые порты будут автоматически пробрасываться, (случайные порты)
52.  `COPY files/index.html /var/www/html/` - копирует файл `files/index.html` внутрь контейнера в папку `/var/www/html/` 
53. `WORKDIR /var/www/html` - рабочая директория, при `exec it` сразу попадаем туда
54.  `COPY files/index.html .` - копирует файл `files/index.html` внутрь контейнера в рабочую папку
55. `ENV TYPE=demo` - переменные среды, внутри контейнера будет доступна данная переменная
56. `docker run -d --rm --name mydocker2 -e TYPE=type2 myenv:v01 ` - как присвоить другое значение для переменной, по сравнению с тем значением, которое присваивается в Dockerfile, или ДОБАВИТЬ новую переменную
57. ![[Pasted image 20250223200939.png]]
58. ![[Pasted image 20250307215428.png]]
59. ![[Pasted image 20250307215832.png]]
60. ![[Pasted image 20250307215903.png]]
61. 
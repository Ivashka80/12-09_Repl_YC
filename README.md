# Домашнее задание к занятию «Базы данных в облаке»

<details>

  ### Инструкция по выполнению домашнего задания

1. Сделайте fork [репозитория c шаблоном решения](https://github.com/netology-code/sys-pattern-homework) к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/gitlab-hw или https://github.com/имя-вашего-репозитория/8-03-hw).
2. Выполните клонирование этого репозитория к себе на ПК с помощью команды `git clone`.
3. Выполните домашнее задание и заполните у себя локально этот файл README.md:
   - впишите вверху название занятия и ваши фамилию и имя;
   - в каждом задании добавьте решение в требуемом виде: текст/код/скриншоты/ссылка;
   - для корректного добавления скриншотов воспользуйтесь инструкцией [«Как вставить скриншот в шаблон с решением»](https://github.com/netology-code/sys-pattern-homework/blob/main/screen-instruction.md);
   - при оформлении используйте возможности языка разметки md. Коротко об этом можно посмотреть в [инструкции по MarkDown](https://github.com/netology-code/sys-pattern-homework/blob/main/md-instruction.md).
4. После завершения работы над домашним заданием сделайте коммит (`git commit -m "comment"`) и отправьте его на Github (`git push origin`).
5. Для проверки домашнего задания преподавателем в личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
6. Любые вопросы задавайте в чате учебной группы и/или в разделе «Вопросы по заданию» в личном кабинете.

Желаем успехов в выполнении домашнего задания.

---

Домашнее задание подразумевает, что вы уже делали предыдущие работы в Яндекс.Облаке, и у вас есть аккаунт и каталог.


Используйте следующие рекомендации во избежание лишних трат в Яндекс.Облаке:
1) Сразу после выполнения задания удалите кластер.
2) Если вы решили взять паузу на выполнение задания, то остановите кластер.

</details>

---

### Задание 1


#### Создание кластера
1. Перейдите на главную страницу сервиса Managed Service for PostgreSQL.
1. Создайте кластер PostgreSQL со следующими параметрами:
- класс хоста: s2.micro, диск network-ssd любого размера;
- хосты: нужно создать два хоста в двух разных зонах доступности и указать необходимость публичного доступа, то есть публичного IP адреса, для них;
- установите учётную запись для пользователя и базы.

Остальные параметры оставьте по умолчанию либо измените по своему усмотрению.

* Нажмите кнопку «Создать кластер» и дождитесь окончания процесса создания, статус кластера = RUNNING. Кластер создаётся от 5 до 10 минут.

![image](https://github.com/Ivashka80/12-09_Repl_YC/assets/121082757/152c8057-70ea-4e2d-bbc6-ada45904c239)

![image](https://github.com/Ivashka80/12-09_Repl_YC/assets/121082757/a17a967a-ec37-4825-ae3b-9cb71335bf92)

#### Подключение к мастеру и реплике 

* Используйте инструкцию по подключению к кластеру, доступную на вкладке «Обзор»: cкачайте SSL-сертификат и подключитесь к кластеру с помощью утилиты psql, указав hostname всех узлов и атрибут ```target_session_attrs=read-write```.

![image](https://github.com/Ivashka80/12-09_Repl_YC/assets/121082757/88490b11-b3ce-4c2f-92f5-df9133752cd8)


* Проверьте, что подключение прошло к master-узлу.
```
select case when pg_is_in_recovery() then 'REPLICA' else 'MASTER' end;
```
![image](https://github.com/Ivashka80/12-09_Repl_YC/assets/121082757/86218dfd-3d0a-4b90-b287-a81ea435e349)

* Посмотрите количество подключенных реплик:
```
select count(*) from pg_stat_replication;
```
![image](https://github.com/Ivashka80/12-09_Repl_YC/assets/121082757/6805d5a2-b46d-466a-b4c0-847675e344f9)

### Проверьте работоспособность репликации в кластере

* Создайте таблицу и вставьте одну-две строки.
```
CREATE TABLE test_table(text varchar);
```
```
insert into test_table values('Строка 1');
```
![image](https://github.com/Ivashka80/12-09_Repl_YC/assets/121082757/dd5a3d9f-86ce-4a61-9292-7c706fc5ec42)

* Выйдите из psql командой ```\q```.

* Теперь подключитесь к узлу-реплике. Для этого из команды подключения удалите атрибут ```target_session_attrs```  и в параметре атрибут ```host``` передайте только имя хоста-реплики. Роли хостов можно посмотреть на соответствующей вкладке UI консоли.

![image](https://github.com/Ivashka80/12-09_Repl_YC/assets/121082757/b579f190-7205-404c-a8fa-2b553be20bca)

* Проверьте, что подключение прошло к узлу-реплике.
```
select case when pg_is_in_recovery() then 'REPLICA' else 'MASTER' end;
```
![image](https://github.com/Ivashka80/12-09_Repl_YC/assets/121082757/e0175aee-4e03-46cd-b348-390e5bb568d7)

* Проверьте состояние репликации
```
select status from pg_stat_wal_receiver;
```
![image](https://github.com/Ivashka80/12-09_Repl_YC/assets/121082757/dc8115b0-1bae-461d-ae83-f81c8a098870)

* Для проверки, что механизм репликации данных работает между зонами доступности облака, выполните запрос к таблице, созданной на предыдущем шаге:
```
select * from test_table;
```
![image](https://github.com/Ivashka80/12-09_Repl_YC/assets/121082757/b6c360d0-ab4c-4792-abd1-95e89cd73dc0)

*В качестве результата вашей работы пришлите скриншоты:*

*1) Созданной базы данных;*
*2) Результата вывода команды на реплике ```select * from test_table;```.*

---

<details>

### Задание 2*

Создайте кластер, как в задании 1 с помощью Terraform.


*В качестве результата вашей работы пришлите скришоты:*

*1) Скриншот созданной базы данных.*
*2) Код Terraform, создающий базу данных.*

---

Задания, помеченные звёздочкой, — дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.

</details>

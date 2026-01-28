# Домашнее задание к занятию "`Защита хоста`" - `Юрочкин В.А.`


---

### Задание 1

Установите eCryptfs.
Добавьте пользователя cryptouser.
Зашифруйте домашний каталог пользователя с помощью eCryptfs.
В качестве ответа пришлите снимки экрана домашнего каталога пользователя с исходными и зашифрованными данными.

`Решение:`

1. Установка eCryptfs

```
apt install -y ecryptfs-utils
```

Проверка:

```
which ecryptfs-migrate-home
```

<img width="526" height="75" alt="image" src="https://github.com/user-attachments/assets/64dcbb46-2cec-413b-ac11-0de03af3902e" />


2. Добавление пользователя cryptouser

```
adduser cryptouser
```

Проверим:

```
id cryptouser
ls -ld /home/cryptouser
```

<img width="851" height="508" alt="image" src="https://github.com/user-attachments/assets/8f91c0ee-6b61-46e5-99c3-05dbdcda5d27" />

3. Создание тестовых исходных данных (до шифрования)

Зайди под пользователем:

```
su - cryptouser
```

Создай файлы:

```
mkdir ~/docs
echo "SECRET DATA 1" > ~/docs/file1.txt
echo "TOP SECRET 2" > ~/docs/file2.txt
ls -l ~/docs
cat ~/docs/file1.txt
```

<img width="748" height="263" alt="image" src="https://github.com/user-attachments/assets/f322d2ca-82f3-4c2c-a3f4-6bd2e4b61350" />

Выйди из пользователя:

```
exit
```

4. Шифрование домашнего каталога cryptouser

Запускаем миграцию:

```
ecryptfs-migrate-home -u cryptouser
```
<img width="1496" height="952" alt="image" src="https://github.com/user-attachments/assets/eacdc402-9935-4080-ad93-9a850ecf2866" />

5. Проверка зашифрованных данных (из-под root)

Смотрим как выглядит каталог без расшифровки:

```
ls -l /home/cryptouser
ls -l /home/cryptouser/.Private
ls -l /home/cryptouser/.ecryptfs
```

<img width="1561" height="262" alt="image" src="https://github.com/user-attachments/assets/bf8c6d5d-f1a6-4ab2-8f18-adf1dc868606" />


Попробую посмотреть файл напрямую:

```
cat /home/cryptouser/.Private/docs/file1.txt
```

<img width="854" height="83" alt="image" src="https://github.com/user-attachments/assets/66dfd9df-419c-400b-9b8e-3b1151f23ac5" />

Смотрим что зашифровано:

```
ls -l /home/.ecryptfs/cryptouser/.Private
```

<img width="1566" height="379" alt="image" src="https://github.com/user-attachments/assets/876370fa-89fb-4271-a864-0da7b21eb1c2" />


6. Проверка после входа пользователя (расшифровка)

Теперь логинимся под пользователем:

```
su - cryptouser
```


При входе:

система автоматически расшифрует home

Проверка:


```
ls -l ~/docs
cat ~/docs/file1.txt
```

<img width="718" height="211" alt="image" src="https://github.com/user-attachments/assets/9c04a617-56e2-43a3-b3f7-3d1c4a1c9626" />

---

### Задание 2
Установите поддержку LUKS.
Создайте небольшой раздел, например, 100 Мб.
Зашифруйте созданный раздел с помощью LUKS.
В качестве ответа пришлите снимки экрана с поэтапным выполнением задания.


`Решение:`

1. Установка поддержки LUKS (cryptsetup)

```
apt install -y cryptsetup
```

Проверка:

```
cryptsetup --version
```

<img width="1259" height="233" alt="image" src="https://github.com/user-attachments/assets/bdd4bfa9-5b42-4abb-8623-207db3304dcd" />

2. Создание небольшого раздела 100 МБ

```
dd if=/dev/zero of=/root/luks_disk.img bs=1M count=100
```

Проверка:

```
ls -lh /root/luks_disk.img
```

<img width="808" height="162" alt="image" src="https://github.com/user-attachments/assets/e02f2a37-3307-4b4e-982a-c9da8d9099cd" />

2.1 Подключаем файл как блочное устройство

```
losetup -fP /root/luks_disk.img
```

Узнать, какое loop-устройство выдано:

```
losetup -a
```

<img width="773" height="156" alt="image" src="https://github.com/user-attachments/assets/47feba11-6f27-4c2f-91c8-d091e3f616cd" />

3. Инициализация LUKS

```
cryptsetup luksFormat /dev/loop3
```

<img width="641" height="240" alt="image" src="https://github.com/user-attachments/assets/3c47473b-64bc-42b7-b7bb-04c5aea1c8a0" />


4. Открываем зашифрованный раздел

```
cryptsetup open /dev/loop3 luks_test
```

Проверка:

```
ls -l /dev/mapper/
```

<img width="901" height="235" alt="image" src="https://github.com/user-attachments/assets/1a21fbed-bc49-4e73-bbea-7e66d61df6ee" />

5. Создание файловой системы

```
mkfs.ext4 /dev/mapper/luks_test
```

<img width="741" height="225" alt="image" src="https://github.com/user-attachments/assets/b0150a49-ccff-4bba-bdb4-68e51a5cb2d3" />

6. Монтирование

```
mkdir /mnt/luks
mount /dev/mapper/luks_test /mnt/luks
```

Проверка:

```
df -h | grep luks
```

<img width="786" height="131" alt="image" src="https://github.com/user-attachments/assets/98f39e87-b238-49e1-b026-c873fcfe4ac4" />


7. Проверка работы

```
echo "LUKS SECRET DATA" > /mnt/luks/secret.txt
cat /mnt/luks/secret.txt
```

<img width="725" height="97" alt="image" src="https://github.com/user-attachments/assets/a3408071-88c9-4872-a02b-4a03397efa6b" />

8. Демонстрация защиты (очень важно для оценки)

```
umount /mnt/luks
cryptsetup close luks_test
```

Проверка:

```
ls /mnt/luks
```

или

```
mount /dev/loop3 /mnt/luks
```

Без cryptsetup open:

ядро видит только зашифрованный контейнер

файловая система недоступна

<img width="676" height="138" alt="image" src="https://github.com/user-attachments/assets/f37250cc-afc5-4aab-8e32-81bf4f6441f9" />








__TOC__

= Переименовать файлы =
# Добавить префикс к именам файлов
Добавить префекс "20" ко всем именам  файлов
*sleep 1; find .  -type f -maxdepth 1 -print0 | xargs -0n 1 bash -c 'preffix="20"; f=$(dirname "$0")/$(basename "$0"); n=$(basename "$0"); n="$preffix$n"; n=$(dirname "$0")/$n; if ["$f" != "$n" ](); then mv --verbose --no-clobber --no-target-directory "$f" "$n"; echo mv -v \"$n\" \"$f\" >>anti.mv; else echo "  -- skipping \"$f\"" >/dev/null; fi; chmod +x anti.mv; '

# Добавить постфикс к именам файлов
Добавить постфикс "-a" ко всем именам  файлов
*sleep 1; find .  -type f -maxdepth 1 -print0 | xargs -0n 1 bash -c 'postffix="-a"; f=$(dirname "$0")/$(basename "$0"); n=$(basename "$0"); extension="${n##*.}"; namewoextension="${n%.*}"; n="${namewoextension}$postffix.${extension}"; n=$(dirname "$0")/$n; if ["$f" != "$n" ](); then mv --verbose --no-clobber --no-target-directory "$f" "$n"; echo mv -v \"$n\" \"$f\" >>anti.mv; else echo "  -- skipping \"$f\"" >/dev/null; fi; chmod +x anti.mv; '

# Удалить несколько символов из начала и несколько символов из конца имени файла
*Почти работает, добавляет лишнее расширение
echo 'Отрицательный $offset отрезает от конца, положительный - от начала'; >/dev/null; find . -type f -maxdepth 1 -print0 | xargs -0n 1 bash -c 'f=$(dirname "$0")/$(basename "$0"); n=$(basename "$0"); extension="${n##*.}"; namewoextension="${n%.*}"; offset=-9; namewoextension=${n:($offset)}; n="${namewoextension}.${extension}"; n=$(dirname "$0")/$n; if ["$f" != "$n" ](); then mv --verbose --no-clobber --no-target-directory "$f" "$n"; echo mv -v \"$n\" \"$f\" >>anti.mv; else echo " -- skipping \"$f\"" >/dev/null; fi; chmod +x anti.mv; '

# Добавить цифровой порядковый возрастающий префикс к именам файлов
<pre>
j=1; \
echo "$j" > ../iterator.value; \
j=$(cat ../iterator.value); \
find . -type f -maxdepth 1 -print0 | \
sort -z | \
xargs -0n 1 bash -c '\
j=$(cat ../iterator.value); \
j=$((j+1)); \
echo "$j" > ../iterator.value; \
preffix=`printf "%05d" $j`; \
preffix="000000000000${j}"; \
preffix="${preffix:(-6)}-"; \
f=$(dirname "$0")/$(basename "$0"); \
n=$(basename "$0"); \
n="$preffix$n"; \
n=$(dirname "$0")/$n; \
if ["$f" != "$n" ](); \
then mv --verbose --no-clobber --no-target-directory "$f" "$n"; \
echo mv -v \"$n\" \"$f\" >>../anti-number.mv; \
else echo " -- skipping \"$f\"" >/dev/null; \
fi; \
chmod +x ../anti-number.mv; '; \
mv -v ../anti-number.mv .; \
rm -v ../iterator.value;
</pre>

## Добавить цифровой возрастающий префикс и создать жёсткие ссылки на те же файлы, но в обратном порядке
<pre>
j=0; \
echo "$j" > ../iterator.value; \
j=$(cat ../iterator.value); \
find . -maxdepth 1 -type f | tr '\0' '\0\n' | wc -l > ../iterator.max; \
find . -maxdepth 1 -type f -print0 | sort -z | \
xargs -0n 1 bash -c '\
max_iterator=$(cat ../iterator.max); \
j=$(cat ../iterator.value); \
j=$((j+1)); \
echo "$j" > ../iterator.value; \
j2=$(( max_iterator*2 - j + 1)); \
preffix="000000000000${j}"; \
preffix="${preffix:(-6)}-"; \
preffix2="000000000000${j2}"; \
preffix2="${preffix2:(-6)}-"; \
f=$(dirname "$0")/$(basename "$0"); \
n=$(basename "$0"); \
n2="$preffix2$n"; \
n="$preffix$n"; \
n2=$(dirname "$0")/$n2; \
n=$(dirname "$0")/$n; \
if ["$f" != "$n" ](); \
then \
mv --verbose --no-clobber --no-target-directory "$f" "$n"; \
ln -v "$n" "$n2"; \
echo rm -v \"$n2\" >>../anti-number.mv; \
echo mv -v \"$n\" \"$f\" >>../anti-number.mv; \
else echo " -- skipping \"$f\"" >/dev/null; \
fi; \
chmod +x ../anti-number.mv; '; \
mv -v ../anti-number.mv .; \
rm -v ../iterator.value; rm -v ../iterator.max;
</pre>

## Добавить цифровой возрастающий префикс к именам файлов по модулю 5
Каждый пятый файл будет иметь префикс "05". modulus задаётся в коде
*<nowiki>sleep 1; j=1; echo "$j" > ../iterator.value; j=$(cat ../iterator.value); find . -type f -maxdepth 1 -print0 | sort -z | xargs -0n 1 bash -c 'j=$(cat ../iterator.value); modulus=5; j=$(($j % $modulus)); j=$((j+1)); echo "$j" > ../iterator.value; preffix=`printf "%02d" $j`; preffix="${preffix}-" f=$(dirname "$0")/$(basename "$0"); n=$(basename "$0"); n="$preffix$n"; n=$(dirname "$0")/$n; if ["$f" != "$n" ](); then mv --verbose --no-clobber --no-target-directory "$f" "$n"; echo mv -v \"$n\" \"$f\" >>../anti-number.mv; else echo " -- skipping \"$f\"" >/dev/null; fi; chmod +x ../anti-number.mv; '; rm -v ../iterator.value; mv -v ../anti-number.mv .;</nowiki>

# Добавить случайный префикс к именам файлов
sleep 1; find . -type f -maxdepth 1 -print0 | xargs -0n 1 bash -c 'preffix=$(($RANDOM % 10))$(($RANDOM % 10))$(($RANDOM % 10))$(($RANDOM % 10))$(($RANDOM % 10))$(($RANDOM % 10))$(($RANDOM % 10)); f=$(dirname "$0")/$(basename "$0"); n=$(basename "$0"); n="$preffix-$n"; n=$(dirname "$0")/$n; if ["$f" != "$n" ](); then mv --verbose --no-clobber --no-target-directory "$f" "$n"; echo mv -v \"$n\" \"$f\" >>anti-random.mv; else echo " -- skipping \"$f\"" >/dev/null; fi; chmod +x anti-random.mv; '

# Добавить начальные нули к имени файлов
find . -type f -maxdepth 1 -print0 | xargs -0n 1 bash -c 'preffix="000000000000000000"; f=$(dirname "$0")/$(basename "$0"); n=$(basename "$0"); n="$preffix$n"; n=${n:(-9)}; n=$(dirname "$0")/$n; if ["$f" != "$n" ](); then mv --verbose --no-clobber --no-target-directory "$f" "$n"; echo mv -v \"$n\" \"$f\" >>antipad.mv; else echo " -- skipping \"$f\"" >/dev/null; fi; chmod +x antipad.mv; '
# Добавить md5-сумму к имени файлов
find . -type f -maxdepth 1 -print0 | xargs -0n 1 bash -c 'checksum=`md5sum "$0"`; checksum=$( echo $checksum | awk "{ print \$1 }"; ); echo $checksum; preffix="${checksum}-"; f=$(dirname "$0")/$(basename "$0"); n=$(basename "$0"); n="$preffix$n"; n=$(dirname "$0")/$n; if ["$f" != "$n" ](); then mv --verbose --no-clobber --no-target-directory "$f" "$n"; echo mv -v \"$n\" \"$f\" >>antichecksumrename.mv; else echo " -- skipping \"$f\"" >/dev/null; fi; chmod +x antichecksumrename.mv; '

# Добавить префикс Ширина_в_пикселях_X_высота_в_пикселях к имени изображения
sleep 1; find . -type f -maxdepth 1 -print0 | xargs -0n 1 bash -c 'calc(){ awk "BEGIN { print "$*" }"; }; f=$(dirname "$0")/$(basename "$0"); n=$(basename "$0"); imagew=`convert "$f" -format %w info:`; echo width=$imagew; imageh=`convert "$f" -format %h info:`; echo height=$imageh; if ["$imagew" -gt "0" ]() && ["$imageh" -gt "0" ](); then ratiohw=`calc "${imageh}/${imagew}"`; addix="${ratiohw}_${imagew}_x_${imageh}_";  else addix=""; fi;  n="$n"; n=$(dirname "$0")/"${addix}${n}"; if ["$f" != "$n" ](); then mv --verbose --no-clobber --no-target-directory "$f" "$n"; echo mv -v \"$n\" \"$f\" >>anti.mv; else echo " -- skipping \"$f\"" >/dev/null; fi; chmod +x anti.mv; '

# Добавить дату создания к именам фото
*Работает в случае, если в метаданных фото записаны данные, требует exiftool, создаёт команды для обратного переименования, если в файле не записана дата - оставляет его как есть, не переименовывает
find . -maxdepth 1 -type f  -print0 | xargs -0n 1 bash -c 'f=$(dirname "$0")/$(basename "$0"); n=$(basename "$0"); extension="${n##*.}"; namewoextension="${n%.*}"; exifpostfix=`exiftool -veryShort -DateTimeOriginal -d %Y-%m-%d-%H-%M-%S- -s "$f"`; n="${exifpostfix}${namewoextension}.${extension}"; n=$(dirname "$0")/$n; if ["$f" != "$n" ](); then mv --verbose --no-clobber --no-target-directory "$f" "$n"; echo mv -v \"$n\" \"$f\" >>anti.mv; else echo " -- skipping \"$f\"" >/dev/null; fi; chmod +x anti.mv; '

Посмотреть, какие данные о времени записаны в файле (берётся первый попавшийся файл, который выдаёт "ls -1")
*exiftool -time:all -s "$(ls -1 -p | grep -v / | head -n 1)"
Все файлы в папке
*exiftool -time:all -s *.*

Переименовать файлы по дате создания
*find . -maxdepth 1  -type f  -print0 | xargs -0n 1 bash -c 'f=$(dirname "$0")/$(basename "$0"); n=$(basename "$0"); extension="${n##*.}"; namewoextension="${n%.*}"; exifpostfix=`exiftool -veryShort -FileModifyDate -d %Y-%m-%d-%H-%M-%S- -s "$f"`; n="${exifpostfix}${namewoextension}.${extension}"; n=$(dirname "$0")/$n; if ["$f" != "$n" ](); then mv --verbose --no-clobber --no-target-directory "$f" "$n"; echo mv -v \"$n\" \"$f\" >>anti.mv; else echo " -- skipping \"$f\"" >/dev/null; fi; chmod +x anti.mv; '

Сохранить метаданные из всех файлов в текстовые файлы
*for i in *.jpg; do echo $i; exiftool -s "${i}" > "${i%.*}.txt"; done;

# Имена файлов преобразовать в нижний регистр
rename -f 'y/A-Z/a-z/' *

# Переименует заменит ".JPG" на ".jpg" во всех файлах по маске в текущей папке
*rename "s/.JPG/.jpg/g" *.JPG
Заменит пробел на подчёркивание рекурсивно
*sleep 1; find . -maxdepth 1 -type f -exec rename -v 's/ /_/' {} \;
Заменит все пробельные символы на подчёркивание рекурсивно
*find . -type f -exec rename -v 's/\s+/_/g' {} \;

# Убрать пробелы и другие символы из названий фото
Рекурсивно уберёт пробельные символы, заменит некоторые слова, часто встречающиеся в именах фотографий и видео, создаст скрипт, позволяющий всё отменить
*sleep 1; find . -type f -print0 | xargs -0n 1 bash -c 'f=$(dirname "$0")/$(basename "$0"); n=$(dirname "$0")/$(basename "$0"|perl -npe "s/^[](.)+//|s/\#/_/gi|s/\,/_/gi|s/\+/-/gi|s/\%/_/gi|s/\&/_/gi|s/\{/_/gi|s/\}/_/gi|s/\</_/gi|s/\>/_/gi|s/\*/_/gi|s/\?/_/gi|s/\$/_/gi|s/\!/_/gi|s/\"/_/gi|s/\s+/_/gi|s/IMG_//gi|s/_HHT//gi|s/_HDR//gi|s/VID_//gi|s/PANO_//gi|s/Screenshot_//gi|s/\(/_/gi|s/\)/_/gi|s/\:/-/gi|s/\@/_/gi|s/\.jpeg/\.jpg/gi|s/___//gi|y/A-Z/a-z/"); if ["$f" != "$n" ](); then mv --verbose --no-clobber --no-target-directory "$f" "$n"; echo mv --verbose \"$n\" \"$f\" >>anti.mv; else echo " -- skipping \"$f\"" >/dev/null; fi; chmod +x anti.mv; '

# Переименовать файлы вида '''20180524_074538.jpg''' в '''2018-05-24_07-45-38--20180524_074538.jpg'''
*sleep 1;  find .  -type f -maxdepth 1 -print0 | xargs -0n 1 bash -c 'f=$(dirname "$0")/$(basename "$0"); n=$(basename "$0"); n="${n:0:4}-${n:4:2}-${n:6:2}-${n:9:2}-${n:11:2}-${n:13:2}---$n"; n=$(dirname "$0")/$n; if ["$f" != "$n" ](); then mv --verbose --no-clobber --no-target-directory "$f" "$n"; echo mv -v \"$n\" \"$f\" >>anti.mv; else echo "  -- skipping \"$f\"" >/dev/null; fi; chmod +x anti.mv; '

= Перенести файлы в директории =
# Разделить файлы по папкам так чтобы в каждой папке оказалось не более заданного размера файлов, рекурсивно
Проходимся по всем файлам, начиная с текущей папки. Если после добавления файла в текущую папку её размер не превысит максимальный - переносим в эту папку, иначе создаём новую.

# Перенести файлы в папки по расширению
Разделить все файлы по папкам по расширению. То есть файлы с расширениями mkv окажутся в одном файлов дереве, а с jpg - в другом. При слиянии структуры директорий восстановится исходный порядок. Работает рекурсивно.
*find . -type f -print0 | xargs -0n 1 bash -c 'fdirname=$(dirname "$0"); f=$(dirname "$0")/$(basename "$0"); n=$(basename "$0"); fextension="${n##*.}"; n="$n"; n="$fextension"/$(dirname "$0")/"$n"; if ["$fextension" != "mv" ](); then mkdir -p "$(dirname "$n")"; mv --verbose --no-clobber --no-target-directory "$f" "$n"; echo mv -v \"$n\" \"$f\" >>anti.mv; else echo " -- skipping \"$f\""; fi; chmod +x anti.mv; '
То же, но не сохраняет оригинальную структуру папок
*find . -type f -print0 | xargs -0n 1 bash -c 'fdirname=$(dirname "$0"); f=$(dirname "$0")/$(basename "$0"); n=$(basename "$0"); fextension="${n##*.}"; n="$n"; n="$fextension"/"$n"; if ["$fextension" != "mv" ](); then mkdir -p "$(dirname "$n")"; mv --verbose --no-clobber --no-target-directory "$f" "$n"; echo mv -v \"$n\" \"$f\" >>anti.mv; else echo " -- skipping \"$f\""; fi; chmod +x anti.mv; '

# Записать файлы в текущей директории в случайном порядке
Полезно для перемешивания mp3-файлов на флешках для hardware-проигрывателях
*find . -type f -maxdepth 1 -print0 | sort -Rz | xargs -0n 1 bash -c 'mkdir -p TEMPDIR; mv -v "$0" "TEMPDIR/"; '; cd "TEMPDIR/"; find . -type f -maxdepth 1 -print0 | sort -Rz | xargs -0n 1 bash -c 'mv -v "$0" "../"; '; cd ..; rmdir "TEMPDIR";
В обычном порядке
*time (find . -maxdepth 1111 -type f  -print0 | sort -z | xargs -0n 1 bash -c 'mkdir -p TEMPDIR; cp -v "$0" "TEMPDIR/"; '; cd "TEMPDIR/"; find . -maxdepth 1111 -type f -print0 | sort -z | xargs -0n 1 bash -c 'mv -v "$0" "../"; '; cd ..; rmdir "TEMPDIR";)

# Скопировать несколько случайных файлов во временную директорию (сколько успеет до прерывания пользвателем)
*mkdir -pv TEMPDIR; time find . -maxdepth 77 -type f -not -path "./TEMPDIR/*" -print0 | sort -Rz | xargs -0n 1 bash -c 'sleep 0.1; cp -v "$0" "TEMPDIR/"; ';
*time find . -maxdepth 777 -type f  \( -iname \*.jpg -o -iname \*.jpeg \) -print0 | sort -Rz | xargs -0n 1 bash -c 'cp -v "$0" "/home/i/downloads/2delete/"; ';

= Метки времени изменить =
# Установить дату создания файлов последовательно завтрашней даты, рекурсивно
*<nowiki>j=$(date -u +%s); j=$((j + 7*24*3600)); echo -n "Start_date="; echo "$j" | tee "iterator.value"; j=$(cat iterator.value); find . -type f -maxdepth 1 -print0 | sort -z | xargs -0n 1 bash -c 'j=$(cat iterator.value); j=$((j+67)); echo "$j" > iterator.value; echo "$j" "$(date -d @${j})" "$0"; touch -a -m --date="@${j}" "$0"; '; rm -v "iterator.value";</nowiki>
Если есть несколько уровней иерархии директорий, имена файлов на разных языках, то может давать неожиданные результаты (даты изменения файлов внутри одной папки будут не по порядку)

# Случайное время создания каждого файла
*<nowiki>find . -type f -maxdepth 1111 -print0 | sort -z | xargs -0n 1 bash -c 'j=$(( ${RANDOM} * ${RANDOM} * 5 + ${RANDOM} )); echo "$j" "$(date -d @${j})" "$0"; touch -a -m --date="@${j}" "$0"; '</nowiki>

# Установить дату изменения файла из имени файла
Нужны файлы имеющие имена вида "2019-06-08-09-27-06" (YYYY-MM-DD HH:mm:ss)
*for i in *.*; do echo $i; filedate="${i:0:4}-${i:5:2}-${i:8:2} ${i:11:2}:${i:14:2}:${i:17:2}"; echo "date = ${filedate}"; touch -a -m --date="${filedate}" "${i}"; done;
# Поставить всем файлам заданную дату изменения и доступа
*find . -type f -exec echo touch -a -m --date="2022-09-22 22:22:22"  {} \;



= Добавить автора к данным файла =
*exiftool -Artist='Borodin-Atamanov.ru' -Copyright='Borodin-Atamanov.ru©' -By-line='Borodin-Atamanov.ru' -Credit='Borodin-Atamanov.ru' -Contact='photos@Borodin-Atamanov.ru' '-xmp-xmprights:marked=1' -overwrite_original *.*

Сначала удалить все мета-теги ( -all= ), а потом добавить нужные
*exiftool -all=  -Artist='Borodin-Atamanov.ru' -Copyright='Borodin-Atamanov.ru©' -By-line='Borodin-Atamanov.ru' -Credit='Borodin-Atamanov.ru' -Contact='photos@Borodin-Atamanov.ru' '-xmp-xmprights:marked=1' -overwrite_original *.*

= Удалить файлы и папки =
# Удалить пустые папки и файлы рекурсивно
*find . -size 0 -type f -exec rm -v '{}' \;
*find . -empty -type d -exec rmdir -v '{}' \;
*find . -empty -type d -exec rmdir -v '{}' \;

# Удалить файлы, содержащие в имени строку
*find .   -name "*_thm.*" -type f -exec rm -v '{}' \;
*find .  -name "*.THM" -type f -exec  rm -v '{}' \;
*find .  -name "*_thumb*" -type f -exec  rm -v '{}' \;

# Удалить самые старые файлы
Удаляет 3 старых файла!
*TAB=$'\t'; find . -type f -print0 | xargs -0 /bin/busybox stat -c "%y${TAB}%n" | sort -n | cut -f2- | head -n 3 | xargs -r rm -v

# Удалить неиспользуемые ядра Linux
*dpkg -l linux-* | awk '/^ii/{ print $2}' | grep -v -e `uname -r | cut -f1,2 -d"-"` | grep -e [| grep -E "(image|headers)" | xargs sudo apt-get -y purge

[[Категория:Как](0-9])]
[[[Категория:rename]([Категория:Linux]])]
[[[Категория:Переименование]([Категория:Файлы]])]
[[[Категория:Текст]([Категория:Bash]])]
[[[Категория:Поиск и замена]([Категория:Автоматизация]])]
[выражения]([Категория:Регулярные)]
[[Категория:Regexp]]


## Попробовать

  - ffmpeg -hide\_banner -i stereo-norm.wav -af silencedetect=n=-5dB:d=1
    -f null -

Удалить из видео все отрезки с тишиной

  - <http://ffmpeg.org/ffmpeg-filters.html#deshake>
  - <http://ffmpeg.org/ffmpeg-filters.html#mestimate> Estimate and
    export motion vectors using block matching algorithms. Motion
    vectors are stored in frame side data to be used by other filters.

### Шумы убрать

  - <http://ffmpeg.org/ffmpeg-filters.html#sab>
  - <http://ffmpeg.org/ffmpeg-filters.html#hqdn3d-1> This is a high
    precision/quality 3d denoise filter. It aims to reduce image noise,
    producing smooth images and making still images really still. It
    should enhance compressibility
  - <http://ffmpeg.org/ffmpeg-filters.html#atadenoise>
  - <http://ffmpeg.org/ffmpeg-filters.html#bm3d> Denoise frames using
    Block-Matching 3D algorithm
  - <http://ffmpeg.org/ffmpeg-filters.html#dctdnoiz>
  - <http://ffmpeg.org/ffmpeg-filters.html#fftdnoiz>

### Аудиокомпрессия

  - <http://ffmpeg.org/ffmpeg-filters.html#acompressor> аудио компрессия
  - <http://ffmpeg.org/ffmpeg-filters.html#acontrast>
  - <http://ffmpeg.org/ffmpeg-filters.html#compand>
  - <http://ffmpeg.org/ffmpeg-filters.html#dynaudnorm> интересный
    вариант

## Как установить ffmpeg в ubuntu 14.04

Устанавливаю ffmpeg.

~~`sudo``   ``add-apt-repository``   ``ppa:jon-severinsson/ffmpeg`~~  
`sudo add-apt-repository ppa:joyard-nicolas/ffmpeg`  
`sudo apt-get update`  
`sudo apt-get install ffmpeg`

## Windows

Скачать с <http://ffmpeg.zeranoe.com/>

## Как установить ffmpeg в ubuntu 15.04

Установиться avconv

`sudo apt-get install ffmpeg`

Может быть

`apt-get install ffmpeg-set-alternatives`

## Посмотреть потоки во всех файлах в текущей директории

time for i in \*.\*; do echo "$i"; ffmpeg -hide\_banner -i "$i" 2\>&1 |
grep "Stream"; pause; done;

## Как сделать dvdrip

  - sleep 1; format="mkv"; mkdir -p "$format"; time for i in \*.\*; do
    echo $i; time ffmpeg -y -hide\_banner -i "$i" -vf
    yadif,mcdeint=mode=extra\_slow:parity=1:qp=10,setsar=1:1,hqdn3d,format=pix\_fmts=yuv420p\\|yuvj420p
    -vcodec libx264 -preset veryslow -crf 19 -profile:v baseline -c:a
    copy -map 0:v? -map 0:a? -map 0:s? -c:s subrip -movflags faststart
    -max\_muxing\_queue\_size 4096 "$format/${i%.\*}.$format"; done;

## Сохранять видео с сетевой web-камеры

NEWDIR="$HOME/screencasts/"; mkdir -p "$NEWDIR"; cd "$NEWDIR"; while
true; do sleep 1; timeout 7200 ffmpeg -rtsp\_transport tcp -y -i
<rtsp://brva.ru:45004/unicast> -vf format=pix\_fmts=yuv420p\\|yuvj420p
-vf null -af anull,loudnorm=I=-9,aresample=48000 -profile:v baseline
-vcodec libx264 -preset veryfast -crf 27 -c:a aac -b:a 80k -movflags
faststart -f segment -segment\_time 15:00 -reset\_timestamps 1
${NEWDIR}$(date +%Y-%m-%d-%H-%M-%S)-$(printf "%06d" $(($(find . -type f
-name "\*.\*" | wc -l)+1)))-screencast-%06d.mkv; done;

## Как сохранять изображение с web камеры в linux

Писать видео с камеры

  - fps=30; ffmpeg\_path='ffmpeg'; NEWDIR="$HOME/screencasts/"; mkdir -p
    "$NEWDIR"; cd "$NEWDIR"; sleep 0; time timeout 12345 ${ffmpeg\_path}
    -hide\_banner -f v4l2 -input\_format mjpeg -framerate ${fps} -i
    /dev/video0 -video\_size 1280x720 -vcodec copy -f segment
    -segment\_time 00:01:00 -reset\_timestamps 1 -movflags faststart
    $NEWDIR$(printf "%06d" $(($(find . -type f -name "\*.\*" | wc
    -l)+1)))-screencast.mkv

<!-- end list -->

  - time ffmpeg -hide\_banner -y -s 3840x2160 -f video4linux2 -i
    /dev/video0 -vframes 1 -vf pp=ac/al/dr/ha/va,unsharp -q:v 0
    webcam.jpg

Записать изображение с камеры как есть - простейший вариант

`avconv -f video4linux2  -s 1280x720 -i /dev/video0 img.jpg;`

Записывать изображение каждую секунду 1000 раз - получать с камеры в
формате 1280x720, потом сжимать до 640x360 и сохранять в jpg. Делать
так каждые 2 секунды

``time for i in `seq 1 1000`; do echo $i; avconv -f video4linux2  -s 1280x720 -i /dev/video0 -s 640x360 `date +%Y-%m-%d-%H-%M-%S`-$i.jpg; sleep 2; done;``

Делать снимок с камеры и с экрана каждую секунду 1000 раз сохранять в
текущей папке с именем по дате

``time for i in `seq 1 1000`; do echo $i; avconv -f x11grab -s 1366x768 -i :0.0 -s 640x360 `date +%Y-%m-%d-%H-%M`-$i-cmr.jpg; avconv -f video4linux2 -s 1280x720 -i /dev/video0 -s 640x360 `date +%Y-%m-%d-%H-%M`-$i-scr.jpg; sleep 1; done;``

Делать снимок с камеры и снимок экрана каждые десять секунд совмещать их
и сохранять каждые 10 секунд с качеством 77%

``time for i in `seq 1 100000`; do echo $i; avconv -f x11grab -s 1366x768 -i :0.0 -s 640x360 /media/data/save2delete/$i-cmr.png; avconv -f video4linux2 -s 1280x720 -i /dev/video0 -s 640x360 /media/data/save2delete/$i-scr.png; composite /media/data/save2delete/$i-scr.png -quality 65 -compose multiply -blend 70x30 /media/data/save2delete/$i-cmr.png /media/data/save2delete/`date +%Y-%m-%d-%H-%M`-$i-multi70x30.jpg; rm /media/data/save2delete/$i-cmr.png; rm /media/data/save2delete/$i-scr.png; sleep 10; done;``

Вычислить различие между двумя изображениями

`convert 3.jpg 2a.jpg -compose Difference -composite -colorspace gray -format '%[mean]' info:`

Различие между двумя изображениями в процентах

`convert 2.jpg 2a.jpg -compose Difference -composite -colorspace gray -format '%[fx:mean*100]' `<info:\>

## Делать скриншот каждые несколько секунд

Сохранять скриншот каждые несколько секунд. Но если изменения между
прошлыми скриншотами меньше заданной величины - удалить последний
из них (то есть не сохранять скриншоты, когда на экране меняется
меньше заданной площади пикселей)

  - <nowiki>

NEWDIR="$HOME/screenshots/"; CUR\_RESOLUTION=\`xdpyinfo | grep
'dimensions:'|awk '{print $2}'\`; echo "$CUR\_RESOLUTION - $NEWDIR";
min\_diff=0.997; mkdir -p $NEWDIR; cd $NEWDIR; while true; do sleep 2;
echo ""; echo "\`date +%Y-%m-%d-%H-%M-%S\`"; next\_file=$(($(find .
-type f -name "\*-screen.png" | wc -l)+1)); next\_file="$(printf %010d
$next\_file)"; next\_diff\_file="$next\_file-diff.ansi";
next\_file="$next\_file-screen.png"; echo $next\_file; ffmpeg -n
-nostats -loglevel 0 -f x11grab -s $CUR\_RESOLUTION -i :0.0 $next\_file;
diff\_command="compare -metric NCC $prev\_file $next\_file -compose src
QQQ-diff.png"; difference=$(($diff\_command) 2\>&1); img2txt
"QQQ-diff.png" \> $next\_diff\_file; cat $next\_diff\_file; echo -n "";
echo mv "QQQ-diff.png" "diff-$difference.png"; if ["$prev\_file" \!=
"$next\_file"]("$prev_file"_!=_"$next_file" "wikilink") && [ bc)"
](1_-eq_"$\(echo_"${difference}_\>_${min_diff}" "wikilink") && [-f
"$prev\_file"](-f_"$prev_file" "wikilink") && [-f
"$next\_file"](-f_"$next_file" "wikilink"); then echo -n ""; echo
"Delete $next\_file, 'cause $prev\_file == $next\_file)"; rm
"$next\_file"; sleep 0; else echo "$difference - It's new\! Better than
$min\_diff"; prev\_file=$next\_file; fi; done; time for ansifile in
\*.ansi; do echo $ansifile; cat $ansifile; sleep 0.1; done; </nowiki>

### Сохранять скриншот в jpg

  - fps=0.1; ffmpeg\_path='ffmpeg'; NEWDIR="$HOME/screencasts/";
    CUR\_RESOLUTION=\`xdpyinfo | grep 'dimensions:'|awk '{print $2}'\`;
    echo $CUR\_RESOLUTION; mkdir -p "$NEWDIR"; cd "$NEWDIR"; sleep 1;
    timeout 72000 ${ffmpeg\_path} -r ${fps} -f x11grab -s
    $CUR\_RESOLUTION -i :0.0 -vf null,pp=ac/dr/ha/va -af anull -qscale:v
    31 -f image2 $NEWDIR$(printf "%06d" $(($(find . -type f -name
    "\*.\*" | wc -l)+1)))-screencast-%08d.jpg
  - \-qscale:v 31 (31 - худшее качество, 0 - лучшее)

### Cхранить скриншот в mp4 h264

sleep 1; fps=5; ffmpeg\_path='ffmpeg'; NEWDIR="$HOME/screencasts/";
CUR\_RESOLUTION=\`xdpyinfo | grep 'dimensions:'|awk '{print $2}'\`; echo
$CUR\_RESOLUTION; mkdir -p "$NEWDIR"; cd "$NEWDIR"; set -x\>/dev/null
2\>&1; timeout 77 ${ffmpeg\_path} -r ${fps} -f x11grab -s
$CUR\_RESOLUTION -i :0.0 -vf
fps=$fps,setsar=1:1,scale=w=-16:h='trunc(min(2160\\,iw)/2)\*2',setsar=1:1,smartblur,normalize=blackpt=black:whitept=white:smoothing=0:independence=1,format=pix\_fmts=yuv420p\\|yuvj420p,fps=$fps
-frames:v 1 -crf 33 -preset veryslow -profile:v main -vcodec h264 -map
0:v? -movflags faststart $NEWDIR\`date "+%F-%H-%M-%S"\`-$(printf "%06d"
$(($(find . -type f -name "\*.\*" | wc -l)+1)))-screenshot.mp4

### Скриншот одной кнопкой через scrot

    #!/bin/bash
    SCREENSHOTDIR="/mnt/ssdata/screeshots/";
    SCREENSHOTDIRDIFF="$SCREENSHOTDIR/diffs/";
    FILESUFFIX="-screen.png";
    mkdir -p $SCREENSHOTDIRDIFF;
    cd $SCREENSHOTDIR;
    #Прошлый файл = количество файлов заданной маски
    prev_file=$(($(find . -type f -name "*$FILESUFFIX" | wc -l)));
    #Следующий файл = прошлый файл + 1
    next_file=$(($prev_file+1));
    #Добавлям leading-zeros
    next_file="$(printf %010d $next_file)";
    next_file_only_number=$next_file
    next_file="$next_file$FILESUFFIX";
    prev_file="$(printf %010d $prev_file)";
    prev_file="$prev_file$FILESUFFIX";
    #Делаем новый скриншот
    #scrot "/mnt/ssdata/screeshots/$next_file";
    #gnome-screenshot --include-pointer --file="/mnt/ssdata/screeshots/$next_file";
    
    if [ $1 -gt 1 ]
    then
    #echo Hey that\'s a large number.
    gnome-screenshot --include-pointer --file="/mnt/ssdata/screeshots/$next_file";
    else
    scrot "/mnt/ssdata/screeshots/$next_file";
    #pwd
    fi
    
    #Вычисляем разницу между изображениями
    diff_command="compare -metric NCC $prev_file $next_file -compose src QQQ-diff.png";
    difference=$(($diff_command) 2>&1);
    #Переносим файл с разницой в папку файлов с разницей, записываем значение разницы
    mv "QQQ-diff.png" "$SCREENSHOTDIRDIFF/$next_file_only_number-diff-$difference.png";
    echo $prev_file;
    echo $next_file;
    echo "[Desktop Entry]" >"$SCREENSHOTDIR/.directory"
    echo "Icon=process-idle-kde" >>"$SCREENSHOTDIR/.directory"

## Как преобразовать в mkv или mp4 без перекодирования

  - sleep 1; format="mp4"; mkdir -p "$format"; time for i in \*.\*; do
    ffmpeg -hide\_banner -i "$i" -c:v copy -c:a copy -map 0:v? -map 0:a?
    -map 0:t? -map 0:d? -map 0:s? -c:s mov\_text -movflags faststart
    "$format/${i%.\*}.$format"; done;
  - sleep 1; format="mkv"; mkdir -p "$format"; time for i in \*.\*; do
    ffmpeg -hide\_banner -i "$i" -c:v copy -c:a copy -map 0:v? -map 0:a?
    -map 0:t? -map 0:d? -map 0:s? -c:s subrip -movflags faststart
    "$format/${i%.\*}.$format"; done;

### Преобразовать в png, jpeg

  - format="png"; mkdir -p "$format"; time for i in \*.\*; do echo "$i";
    mkdir -p "${format}"; ffmpeg -n -hide\_banner -i "${i}" -vframes 1
    -vf null,pp=ac/dr/ha/va -q:v 0 "${format}"/${i%.\*}.${format}; done;
  - format="png"; mkdir -p "$format"; time for i in \*.\*; do echo "$i";
    convert "$i" -quality 100 "$format/${i%.\*}.$format"; done;
  - format="jpg"; mkdir -p "$format"; time for i in \*.\*; do echo "$i";
    convert "$i" -quality 97 "$format/${i%.\*}.$format"; done;

### Конвертировать WebP в jpg

  - format="jpg"; mkdir -p "$format"; time for i in \*.webp; do echo
    "$i"; convert "$i" -quality 100 "${i%.\*}.$format"; done;

## Как перекодировать avi в mkv

sleep 1; format="mkv"; mkdir -p "$format"; time for i in \*.\*; do echo
$i; time ffmpeg -y -hide\_banner -i "$i" -af
"anull,loudnorm=I=-12,aresample=48000" -vf
null,pp=ac/dr/ha/va,format=pix\_fmts=yuv420p\\|yuvj420p -vcodec libx264
-preset veryslow -crf 22 -ac 2 -c:a aac -b:a 128k -map 0:v? -map 0:a?
-map 0:s? -c:s subrip -movflags faststart -max\_muxing\_queue\_size 4096
"$format/${i%.\*}.$format"; done;

## Как нарезать видео на скриншоты в Linux

`time ffmpeg -hide_banner -i video.mp4 -an -r 1/60 -f image2 00out%09d.png`

Сохраняет кадр каждые 60 секунд из видео в файлы, нумерованные по
порядку.

Если изменить 1/60 на 1 - то будет 1 кадр в секунду, если на 30 - будет
сохраняться 30 кадров в секунду. 1/500 - 1 кадр раз в 500 секунд видео
времени.

## Создать статичные видео из изображений или из первых кадров видео

### Несколько кадров

format="mp4"; fps="1.5"; frames=63; crf=42;
OUTDIR="$format-${fps}fps-${frames}frames-${crf}crf"; mkdir -p
"$OUTDIR"; time for i in \*.\*; do time ffmpeg -y -hide\_banner
-loglevel debug -r $fps -i "$i" -an -vf
setsar=1:1,pad=width='ceil(iw/2)\*2':height='ceil(ih/2)\*2',scale=h='trunc(min(2160\\,ih)/2)\*2':w=-2,setsar=1:1,loop=loop=$frames:size=1:start=0,fps=$fps,mpdecimate,format=pix\_fmts=yuv420p\\|yuvj420p,fps=$fps
-frames:v $frames -crf $crf -preset veryslow -profile:v high -vcodec
h264 -x264-params
min-keyint=31337:keyint\_min=31337:keyint=31337:scenecut=31337:ref=1
-movflags faststart "$OUTDIR/${i%.\*}.$format"; done;

### Один кадр

  - format="mp4"; fps="2"; mkdir -p "$format"; time for i in \*.\*; do
    time ffmpeg -y -hide\_banner -r $fps -i "$i" -an -vf
    loop=loop=2:size=1:start=0,fps=$fps,setsar=1:1,scale=h='trunc(min(2160\\,ih)/2)\*2':w=-2,setsar=1:1,format=pix\_fmts=yuv420p\\|yuvj420p,fps=$fps
    -frames:v 1 -crf 35 -preset veryslow -profile:v main -vcodec h264
    -movflags faststart "$format/${i%.\*}.$format"; done;

### Цикл по разному качеству

    NEWDIR="quality-va"; mkdir -pv $NEWDIR; \
    format="mp4"; \
    time for f in *.*; do echo $f; \
    for qua in `seq 0 1 70`; do echo $qua; \
    quality="0000000${qua}"; quality=${quality:(-3)}; \
    fps="0.21"; mkdir -p "$format"; time ffmpeg -y -hide_banner -r $fps -i "$f" -an -vf loop=loop=2:size=1:start=0,fps=$fps,setsar=1:1,scale=h='trunc(min(2160\,ih)/2)*2':w=-2,setsar=1:1,format=pix_fmts=yuv420p\|yuvj420p,fps=$fps -frames:v 1 -crf  ${quality} -preset veryslow -profile:v main -vcodec h264 -movflags faststart "${NEWDIR}/${f%.*}-q${quality}.$format"; \
    done; done;

## Как перекодировать в ogg

``time for i in *.wav; do time ffmpeg -y -hide_banner -i "$i" -ab 192000 -ac 1 -acodec libvorbis "`basename "$i" .wav`.ogg"; done;``

Перекодировать все wav файлы в текущей папке в ogg с качеством
192Kbit/sec, 2 канала

`time for i in /media/disk140/upload/music/VA-pop/*.mp3; do time  ffmpeg -y  -hide_banner -i "$i" -ab 32000 -ac 1 -acodec libvorbis "$i".ogg; done;`

1 канал, давать то же имя, только меняя .wav на .ogg

`time for i in *.wav; do time ffmpeg  -hide_banner  -i "$i" -acodec libvorbis -ab 128k -ac 1 "${i%.*}.ogg"; done;`

  - Преобразует все файлы \*.mp3 в файлы \*.mp3.ogg в заданной папке.
    Использует битрейт 32kbps, берёт только один канал,
    перезаписывает существующие файлы

`time for i in /media/data/music/Jazz-radio/*.mp3; do time ffmpeg -y  -hide_banner -i "$i" -ab 32000 -acodec aac -strict experimental "$i".aac; done;`

  - Перекодирует в raw-aac с качеством 32Кбита в секунду, субъективно
    качество ужасное, чуть ли не хуже чем ogg, видимо встроеный кодек
    кодирует плохо

## Как перекодировать в mp3

`convert Дарит-осень-чудеса.wav Дарит-осень-чудеса.mp3 -ab 320000`

1 канал, 320Кбит/сек

  - time for i in \*.wav; do time ffmpeg -hide\_banner -i "$i" -ab 320k
    -ac 1 -acodec libmp3lame "${i%.\*}.mp3"; done;

48kHz, 2 канала

  - time for i in \*.wav; do time ffmpeg -hide\_banner -i "$i" -af
    aresample=48000 -ab 320k -ac 2 -acodec libmp3lame "${i%.\*}.mp3";
    done;

## Crossfade нескольких файлов, кодирование в один

4 файла

    <nowiki>
    ffmpeg -i 0.mp3 -i 1.mp3 -i 2.mp3 -i 3.mp3 -vn \
    -filter_complex "[0][1]acrossfade=d=10:c1=tri:c2=tri[a01]; \
                           [a01][2]acrossfade=d=10:c1=tri:c2=tri[a02]; \
                           [a02][3]acrossfade=d=10:c1=tri:c2=tri" \
    out.mp3
    </nowiki>

### Любое количество файлов

time (DURATION=10; READYDIR="ready"; mkdir -p "$READYDIR"; for i in
\*.\*; do echo "$i"; ffmpeg -y -hide\_banner -i "$i" -to
00:00:${DURATION} -af "aresample=48000,volume=0.0" -acodec pcm\_s32le
"${READYDIR}/base.wav"; break; done; for i in \*.\*; do echo "$i";
ffmpeg -hide\_banner -i "${READYDIR}/base.wav" -i "${i}"
-filter\_complex aresample=48000,acrossfade=d=${DURATION}:c1=tri:c2=tri
-acodec pcm\_s32le "${READYDIR}/base-new.wav"; rm
"${READYDIR}/base.wav"; mv -v "${READYDIR}/base-new.wav"
"${READYDIR}/base.wav"; sleep 0.23; done; ffmpeg -y -hide\_banner -i
"${READYDIR}/base.wav" -af anull,loudnorm=I=-9,aresample=48000 -ab 320k
"${READYDIR}/crossfaded.mp3";)

## Как записать скринкаст в ubuntu

### Создать виртуальную web-камеру в системе, транслировать на неё рабочий стол

Под root:

  - apt install -y v4l2loopback-dkms
  - modprobe v4l2loopback
  - fps=15; ffmpeg\_path='ffmpeg'; NEWDIR="$HOME/screencasts/";
    CUR\_RESOLUTION=\`xdpyinfo | grep 'dimensions:'|awk '{print $2}'\`;
    echo $CUR\_RESOLUTION; time ${ffmpeg\_path} -hide\_banner -r ${fps}
    -f x11grab -s $CUR\_RESOLUTION -i :0.0 -vf
    scale=1280x720,setdar=16:9,setsar=1:1,format=pix\_fmts=yuv420p\\|yuvj420p
    -reset\_timestamps 1 -vcodec rawvideo -threads 0 -f v4l2
    /dev/video$(($(ls -1 /dev/video\* | wc -l) +1))

#### Транслировать файл в бесконечном цикле в камеру

Под root:

  - modprobe v4l2loopback
  - fps=30; ffmpeg\_path='ffmpeg'; NEWDIR="$HOME/screencasts/";
    first\_file\_name=$(ls -1 -p | grep -v / | head -n 1); time
    ${ffmpeg\_path} -re -hide\_banner -r ${fps} -stream\_loop -1 -i
    "${first\_file\_name}" -vf
    scale=1280x720,setdar=16:9,setsar=1:1,fps=${fps},format=pix\_fmts=yuv420p\\|yuvj420p
    -reset\_timestamps 1 -r ${fps} -vcodec rawvideo -threads 0 -f v4l2
    /dev/video2

### Скринкаст в файл

Сразу писать в libx264, Звук wav, yuv420p, делить на файлы каждую
минуту, остановка записи через 3+ часа или при нажатии q fps лучше
выбирать так, чтобы CPU точно справлялся с кодированием

  - fps=5; ffmpeg\_path='ffmpeg'; NEWDIR="$HOME/screencasts/";
    CUR\_RESOLUTION=\`xdpyinfo | grep 'dimensions:'|awk '{print $2}'\`;
    echo $CUR\_RESOLUTION; mkdir -p "$NEWDIR"; cd "$NEWDIR"; sleep 1;
    timeout 12345 ${ffmpeg\_path} -hide\_banner -f alsa -i pulse -r
    ${fps} -f x11grab -s $CUR\_RESOLUTION -i :0.0 -vf
    null,format=pix\_fmts=yuv420p\\|yuvj420p -af anull -profile:v
    baseline -vcodec libx264 -preset ultrafast -crf 20 -ac 2 -acodec
    pcm\_s16le -movflags faststart -f segment -segment\_time 01:00
    -reset\_timestamps 1 $NEWDIR$(printf "%06d" $(($(find . -type f
    -name "\*.\*" | wc -l)+1)))-screencast-%06d.mkv

#### Писать скринкаст только если есть активность пользователя (мышь или клавиатура)

    NEWDIR="$HOME/screencasts/"; mkdir -pv "$NEWDIR"; events_file="$NEWDIR/events.txt"; \
    rm -v "$events_file"; \
    nohup xinput test-xi2 --root > $events_file & \
    NEWDIR="$HOME/screencasts/"; mkdir -pv "$NEWDIR"; events_file="$NEWDIR/events.txt"; \
    sleep 0.3; timeout 3 tail -f "$events_file"; sleep 0.5; \
    no_activity_wait_sec=3591; \
    while true; \
    do fps=0.09311; \
    CUR_RESOLUTION=`xdpyinfo | grep 'dimensions:'| awk '{print $2}'`; \
    screencast_file="$NEWDIR/`date +%Y-%m-%d--%H-%M-%S`.mkv"; \
    echo "$CUR_RESOLUTION -- $screencast_file"; \
    nice -n 19 timeout $(( no_activity_wait_sec * 2 + 11 )) ffmpeg -n -hide_banner -r ${fps} \
    -f x11grab -s $CUR_RESOLUTION -i :0.0 -vf format=pix_fmts=yuv420p\|yuvj420p -t "$no_activity_wait_sec" \
    -profile:v baseline -vcodec libx264 -preset veryfast -tune stillimage -crf 33 -movflags faststart \
    "$screencast_file"; \
    if [ $(($(date +%s) - $(date +%s -r $events_file))) -gt $no_activity_wait_sec ]; \
    then echo -e "\n\n\nNo activity. Delete last file.\n\n\n"; \
    rm -v "$screencast_file"; sleep 3; \
    else echo "activity detected. Let's record next file"; sleep 1; \
    fi; \
    done;

### Звук с устройства по-умолчанию

  - fps=15; ffmpeg\_path='ffmpeg'; NEWDIR="$HOME/screencasts/";
    CUR\_RESOLUTION=\`xdpyinfo | grep 'dimensions:'|awk '{print $2}'\`;
    echo $CUR\_RESOLUTION; mkdir -p "$NEWDIR"; cd "$NEWDIR"; sleep 1;
    timeout 7200 ${ffmpeg\_path} -hide\_banner -r ${fps} -f x11grab -s
    $CUR\_RESOLUTION -i :0.0 -f alsa -i pulse -vf
    pp=ac/dr/ha/va,setdar=16:9,setsar=1:1,format=pix\_fmts=yuv420p\\|yuvj420p
    -af anull -profile:v baseline -vcodec libx264 -preset ultrafast -crf
    20 -ac 2 -c:a aac -b:a 192k -movflags faststart $NEWDIR$(printf
    "%06d" $(($(find . -type f -name "\*.\*" | wc
    -l)+1)))-screencast-%06d.mkv

### Системный звук, рассинхрон

  - fps=5; ffmpeg\_path='ffmpeg'; NEWDIR="$HOME/screencasts/";
    CUR\_RESOLUTION=\`xdpyinfo | grep 'dimensions:'|awk '{print $2}'\`;
    echo $CUR\_RESOLUTION; mkdir -p "$NEWDIR"; cd "$NEWDIR"; sleep 1;
    timeout 7200 ${ffmpeg\_path} -hide\_banner -r ${fps} -f x11grab -s
    $CUR\_RESOLUTION -i :0.0 -f pulse -i
    alsa\_output.pci-0000\_38\_00.6.analog-stereo.monitor -vf
    null,format=pix\_fmts=yuv420p\\|yuvj420p -af anull -profile:v
    baseline -vcodec libx264 -preset ultrafast -crf 20 -ac 2 -c:a aac
    -b:a 192k -movflags faststart $NEWDIR$(printf "%06d" $(($(find .
    -type f -name "\*.\*" | wc -l)+1)))-screencast-%06d.mkv

Тоже, но без звука

  - fps=5; ffmpeg\_path='ffmpeg'; NEWDIR="$HOME/screencasts/";
    CUR\_RESOLUTION=\`xdpyinfo | grep 'dimensions:'|awk '{print $2}'\`;
    echo $CUR\_RESOLUTION; mkdir -p "$NEWDIR"; cd "$NEWDIR"; sleep 1;
    timeout 7200 ${ffmpeg\_path} -hide\_banner -r ${fps} -f x11grab -s
    $CUR\_RESOLUTION -i :0.0 -vf
    null,format=pix\_fmts=yuv420p\\|yuvj420p -af anull -profile:v
    baseline -vcodec libx264 -preset ultrafast -crf 20 -movflags
    faststart -f segment -segment\_time 07:00 -reset\_timestamps 1
    $NEWDIR$(printf "%06d" $(($(find . -type f -name "\*.\*" | wc
    -l)+1)))-screencast-%06d.mkv

Без звука, низкий битрейт и fps

  - fps=1; ffmpeg\_path='ffmpeg'; NEWDIR="$HOME/screencasts/";
    CUR\_RESOLUTION=\`xdpyinfo | grep 'dimensions:'|awk '{print $2}'\`;
    echo $CUR\_RESOLUTION; mkdir -p "$NEWDIR"; cd "$NEWDIR"; sleep 1;
    timeout 72000 ${ffmpeg\_path} -hide\_banner -r ${fps} -f x11grab -s
    $CUR\_RESOLUTION -i :0.0 -vf
    null,format=pix\_fmts=yuv420p\\|yuvj420p -af anull -profile:v
    baseline -vcodec libx264 -preset ultrafast -crf 61 -movflags
    faststart -f segment -segment\_time 10:00:00 -reset\_timestamps 1
    $NEWDIR$(printf "%06d" $(($(find . -type f -name "\*.\*" | wc
    -l)+1)))-screencast-%06d.mkv

Без звука, низкий fps, высокая загрузка CPU при кодировании

  - fps=0.1; ffmpeg\_path='ffmpeg'; NEWDIR="$HOME/screencasts/";
    CUR\_RESOLUTION=\`xdpyinfo | grep 'dimensions:'|awk '{print $2}'\`;
    echo $CUR\_RESOLUTION; mkdir -p "$NEWDIR"; cd "$NEWDIR"; sleep 1;
    timeout 72000 ${ffmpeg\_path} -hide\_banner -r ${fps} -f x11grab -s
    $CUR\_RESOLUTION -i :0.0 -vf
    scale=1280x720,setdar=16:9,setsar=1:1,sab,hqdn3d,format=pix\_fmts=yuv420p\\|yuvj420p
    -af anull -profile:v baseline -vcodec libx264 -preset veryfast -crf
    27 -movflags faststart -f segment -segment\_time 03:00:00
    -reset\_timestamps 1 $NEWDIR$(printf "%06d" $(($(find . -type f
    -name "\*.\*" | wc -l)+1)))-screencast-%06d.mkv

Без звука, низкий битрейт и fps, высокая загрузка CPU при кодировании

  - fps=1; ffmpeg\_path='ffmpeg'; NEWDIR="$HOME/screencasts/";
    CUR\_RESOLUTION=\`xdpyinfo | grep 'dimensions:'|awk '{print $2}'\`;
    echo $CUR\_RESOLUTION; mkdir -p "$NEWDIR"; cd "$NEWDIR"; sleep 1;
    timeout 72000 ${ffmpeg\_path} -hide\_banner -r ${fps} -f x11grab -s
    $CUR\_RESOLUTION -i :0.0 -vf
    hqdn3d,smartblur=luma\_radius=5:luma\_strength=1:luma\_threshold=10,format=pix\_fmts=yuv420p\\|yuvj420p
    -af anull -profile:v baseline -vcodec libx264 -preset veryslow -crf
    31 -movflags faststart -f segment -segment\_time 10:00:00
    -reset\_timestamps 1 $NEWDIR$(printf "%06d" $(($(find . -type f
    -name "\*.\*" | wc -l)+1)))-screencast-%06d.mkv

Сохранение в jpeg

  - fps=1; ffmpeg\_path='ffmpeg'; NEWDIR="$HOME/screencasts/";
    CUR\_RESOLUTION=\`xdpyinfo | grep 'dimensions:'|awk '{print $2}'\`;
    echo $CUR\_RESOLUTION; mkdir -p "$NEWDIR"; cd "$NEWDIR"; sleep 1;
    timeout 72000 ${ffmpeg\_path} -hide\_banner -r ${fps} -f x11grab -s
    $CUR\_RESOLUTION -i :0.0 -vf
    null,format=pix\_fmts=yuv420p\\|yuvj420p -af anull -qscale:v 0 -f
    image2 $NEWDIR$(printf "%06d" $(($(find . -type f -name "\*.\*" | wc
    -l)+1)))-screencast-%06d.jpg

Пишет с битрейт около 100000 kbit/sec. Гигабайт в минуту примерно.
Запись в контейнер MOV. Примерно 10000 kbit/s. Создаёт папку,
пишет файлы по очереди, ждёт 3 секунды перед записью

  - NEWDIR="/mnt/ssdata/screencasts/"; mkdir -p $NEWDIR; cd $NEWDIR;
    sleep 3; ffmpeg -f alsa -i pulse -f x11grab -r 30 -s 1366x768 -i
    :0.0 -acodec pcm\_s16le -vcodec qtrle $NEWDIR$(($(find . -type f
    -name "\*-screencast.mov" | wc -l)+1))-screencast.mov

Запись с кодеком h264, низким битрейтом, 2 кадра в секунду, имя файла из
текущего времени:

  - NEWDIR="/mnt/ssdata/temp2delete/test"; mkdir -p $NEWDIR; ffmpeg -f
    x11grab -r 2 -s 1920x1080 -i :0.0 -vcodec h264 -b:v 100k -r 60
    $NEWDIR/\`date +%Y-%m-%d-%H-%M-%S\`-screencast.mkv

Запись только 1 час, потом остановка записи, сохранение файла на
удалённой машине может работать после разрыва соединения по ssh

  - NEWDIR="/home/d/temp2delete/test2"; mkdir -p $NEWDIR; nohup timeout
    3600 ffmpeg -f x11grab -r 0.5 -s 1366x768 -i :0.0 -vcodec h264 -b:v
    100k -r 60 $NEWDIR/\`date +%Y-%m-%d-%H-%M-%S\`-screencast.mkv &

Сохранять скриншот экрана каждые 10 секунд

  - NEWDIR="/mnt/ssdata/screencasts37"; mkdir -p $NEWDIR; cd $NEWDIR;
    while true; do sleep 10; avconv -f x11grab -s 1920x1080 -i :0.0
    $(($(find . -type f -name "\*-screen.png" | wc -l)+1))-screen.png;
    done;

### Попробовать записать звук с разных устройств, с разным количеством каналов (обычно количество каналов "2" работает)

  - ARRAY\_IDS=\`ffmpeg -hide\_banner -sources pulse\`; ARRAY\_IDS=(
    $ARRAY\_IDS ); IFS=$'\\n' ARRAY\_IDS=($(sort --unique
    \<\<\<"${ARRAY\_IDS\[\*\]}")); unset IFS; echo "${ARRAY\_IDS\[\*\]}"
    | tee -a "DEVICE\_ID.csv"; for DEVICE\_ID in "${ARRAY\_IDS\[@\]}";
    do for CHANNELS in 2; do echo "${DEVICE\_ID}---ac=${CHANNELS}";
    ID=$(echo "${DEVICE\_ID}-${CHANNELS}" | sed
    's/\[^0-9A-Za-z\]\*//g'); echo $ID; command="ffmpeg -hide\_banner -y
    -f pulse -i "${DEVICE\_ID}" -af
    \\"anull,loudnorm=I=-12,aresample=48000\\" -ac ${CHANNELS} -t 5
    \\"${ID}.wav\\""; echo $command; eval $command; done; done;

Сначала получить список всех возможных устройств, попробовать записать
звук с каждого устройства.

#### Посмотреть звуковые устройства-источники записи

  - ffmpeg -hide\_banner -sources

или

  - ffmpeg -hide\_banner -sources pulse

Обычно запись получается \_примерно\_ таким способом: ffmpeg -f pulse -i
alsa\_output.pci-0000\_38\_00.6.analog-stereo.monitor -t 20 cazzo.mp3

### Перекодировать записанный скринкаст для отправки

  - format="mkv"; mkdir -p "$format"; time for i in \*.\*; do echo $i;
    time ffmpeg -y -hide\_banner -i "$i" -af
    "loudnorm=I=-12,aresample=48000" -vf
    pp=ac/dr/ha/va,scale=1920x1080,setdar=16:9,setsar=1:1,sab,hqdn3d,smartblur=luma\_radius=5:luma\_strength=1:luma\_threshold=10,format=pix\_fmts=yuv420p\\|yuvj420p
    -vcodec libx264 -preset veryslow -crf 27 -movflags faststart
    -profile:v baseline -ac 1 -c:a aac -b:a 128k -map 0:v? -map 0:a?
    -map 0:s? -movflags faststart -max\_muxing\_queue\_size 4096
    "$format/${i%.\*}.$format"; done;

Без звука:

  - format="mp4"; mkdir -p "$format"; time for i in \*.\*; do echo $i;
    time ffmpeg -y -hide\_banner -i "$i" -an -vf
    hqdn3d,pp=ac/dr/ha/va,format=pix\_fmts=yuv420p\\|yuvj420p -vcodec
    libx264 -preset veryslow -crf 31 -movflags faststart -profile:v
    baseline -map 0:v? -map 0:s? -movflags faststart
    -max\_muxing\_queue\_size 4096 "$format/${i%.\*}.$format"; done;

#### h265

format="mkv"; mkdir -p "$format"; time for i in \*.\*; do echo $i; time
ffmpeg -y -hide\_banner -i "$i" -af "loudnorm=I=-12,aresample=48000" -vf
pp=ac/dr/ha/va,setdar=16:9,setsar=1:1,sab,hqdn3d,format=pix\_fmts=yuv420p\\|yuvj420p
-vcodec libx265 -preset veryslow -profile:v mainstillpicture -crf 37
-movflags faststart -acodec libvorbis -ab 128k -map 0 -movflags
faststart -max\_muxing\_queue\_size 4096 "$format/${i%.\*}.$format";
done;

#### Использовать фильтра mpdecimate для того, чтобы заменить слишком похожие кадры на dup

  - mpdecimate
  - <https://ffmpeg.org/ffmpeg-filters.html#mpdecimate>

## Транслировать экран по сети

На сервере

  - fps=10; CUR\_RESOLUTION=\`xdpyinfo | grep 'dimensions:'|awk '{print
    $2}'\`; echo $CUR\_RESOLUTION; timeout 7200 ffmpeg -r ${fps} -f
    x11grab -s $CUR\_RESOLUTION -i :0.0 -vf
    scale=w=iw/2:h=ih/2,unsharp,format=pix\_fmts=yuv420p\\|yuvj420p -af
    anull -profile:v baseline -vcodec libx264 -preset ultrafast -crf 40
    -tune zerolatency -movflags faststart -reset\_timestamps 1 -f avi -
    | nc -l -p 12340

На клиенте

  - nc 127.0.0.1 12340 | ffplay -i -

Задержка около 10 секунд получается

### Соединить изображения в видео

Соединить все файлы изображений в текущей директории в видео

  - fps=30; ext=$(ls -1 -p | grep -v / | head -n 1);
    ext="\\\*.${ext\#\#\*.}"; ext=${ext:1}; echo "${ext}"; time ffmpeg
    -hide\_banner -r $fps -f image2 -pattern\_type glob -i "$ext" -r
    $fps -vf
    pp=ac/dr/ha/va,setsar=1:1,pad=width='ceil(iw/2)\*2':height='ceil(ih/2)\*2',scale=w='trunc(iw/2)\*2':h='trunc(ih/2)\*2',setsar=1:1,hqdn3d,format=pix\_fmts=yuv420p\\|yuvj420p
    -vcodec libx264 -preset veryslow -crf 24 -movflags faststart
    -profile:v main -an "z-\`date "+%F-%H-%M-%S"\`-$fps-fps.mp4";

<!-- end list -->

  - \-preset veryfast - быстрое кодирование (хуже качество)
  - \-preset veryslow - медленное кодирование (лучше качество при том же
    размере)
  - \-profile:v main baseline - main - лучше сжатие (но ниже скорость
    кодирование), baseline - хуже сжатие, лучше совместимость с
    разными устройствами, быстрее кодирование
  - \-crf 0 - показатель "качества", 0 - лучшее, \~64 - худшее

### Изменить скорость видео без потерь и дублирования кадров, только изменяя framerate

Исходное видео 30fps - делаем 60 fps - длительность уменьшится в 2 раза

`ffmpeg -i 30fps-ALL.mp4 -vcodec h264 -an -r 60 -filter:v "setpts=0.5*PTS" 60fps-all.mp4`

### Сделать gif из нескольких png

Берёт из текущей папки файлы вида 0001.png, 0002.png, изменяет размер к
640x360, делает смену кадра один раз в 2 секунды

`ffmpeg -r 0.5 -i %04d.png -s 640x360 -r 0.5 output.gif`

## Как выполнять муксинг и демуксинг видео через ffmpeg

## Смуксить все файлы в текущей директории в один в алфавитном порядке

    map=0; maps_str=""; \
    mux_command="time ffmpeg -hide_banner "; \
    time for i in *.*; \
    do echo "$i"; \
    mux_command="${mux_command} -i \"${i}\" "; \
    maps_str="${maps_str} -map $map"; \
    map=$(($map+1));
    done; \
    mux_command="${mux_command} ${maps_str} -c copy -c:v copy -c:a copy -movflags faststart z-muxed.mp4; "; \
    echo "${mux_command}"; eval "${mux_command}";

### 5.1 в stereo: Вынуть заданную аудио-дорожку в аудио-файл, нормализовать её, смуксить в новый видео файл

time ( audio\_id=0; aformat="mkv"; acodecs=" -acodec libvorbis -ab 192k
"; mkdir -p "$aformat"; time for i in \*.\*; do time ffmpeg -threads 32
-n -hide\_banner -i "$i" -vn -map 0:a:${audio\_id}? -ac 2 ${acodecs}
-c:s mov\_text -movflags faststart
"$aformat/${i%.\*}-original.$aformat"; time ffmpeg -threads 32 -n
-hide\_banner -i "$i" -vn -map 0:a:${audio\_id}? -ac 2 -af
loudnorm=I=-10,aresample=48000 ${acodecs} -c:s mov\_text -movflags
faststart "$aformat/${i%.\*}.$aformat"; done; format="mkv"; mkdir -p
"2${format}"; time for i in \*.\*; do ffmpeg -y -threads 32
-hide\_banner -i "$i" -i "$aformat/${i%.\*}.$aformat" -i
"$aformat/${i%.\*}-original.$aformat" -c:v copy -c:a copy -map 0:v:0?
-map 1:a? -map 2:a? -map 0:a? -map 0:s? -metadata:s:a:0 title="stereo
normalize" -metadata:s:a:1 title="stereo original" -movflags faststart
"2${format}/${i%.\*}.$format"; done; )

### Видео из одного файла, из него же все аудиодорожки, кроме английской, субтитры из двух других файлов

  - format="mp4"; mkdir -p "$format"; time for i in \*.\*; do time
    ffmpeg -hide\_banner -i "$i" -i "RUS/${i%.\*}.srt" -i
    "ENG/${i%.\*}.srt" -c:all copy -c:v copy -c:a copy -map 0:v? -map a
    -map -m:language:eng -map 1:s? -map 2:s? -c:s mov\_text -movflags
    faststart "$format/${i%.\*}.$format"; done;

### Видео из одного файла без перекодирования, звук из другого - с перекодированием в aac

  - format="mp4"; mkdir -p "$format"; time for i in \*.\*; do ffmpeg
    -threads 8 -hide\_banner -i "$i" -c:v copy -c:a copy -map 0:v? -map
    0:a? -map 0:s? -ac 1 -af loudnorm=I=-9,aresample=48000 -c:a aac -b:a
    192k -c:s mov\_text -movflags faststart "$format/${i%.\*}.$format";
    done;
  - time ffmpeg -threads 8 -hide\_banner -i 0.mp4 -i 1.wav -c:v copy -af
    aresample=48000 -c:a aac -b:a 192k -map 0:v -map 1:a -map 0:s? -c:s
    subrip -movflags faststart 2.mkv

Как выполнять муксинг и демуксинг видео через ffmpeg в ubuntu?

Взято тут:
<http://cfc.kizzx2.com/index.php/muxing-audio-and-video-with-ffmpeg/>
Simple example: mux an audio with a video file without audio track

`ffmpeg -i audio.mp3 -i video.avi -acodec copy -vcodec copy output.avi`

1.  Daily usage example: mux an audio with a video file \_with\_ an
    existing audio track.
2.  This will replace the AVI file's audio track with the MP3

`ffmpeg -i audio.mp3 -i video-with-audio.avi -map 0:0 -map 1:0 -acodec copy -vcodec copy -shortest output.avi`

Demux (демуксинг): Вынимаем аудиодорожку 1 из файла в отдельный файл.
(Вынимает только аудио-дорожку)

`time ffmpeg  -hide_banner  -i "5.mp4" -vn -acodec copy "5.wav"`

Или:

`cd /mnt/ssdata/Dropbox/tp-wiki/`  
``time for i in *.mkv; do time ffmpeg  -hide_banner  -i "$i" "`basename "$i" .mp4`.wav" -acodec pcm_s32le; done;``

Или через башизм:

  - DIR='wav'; mkdir -pv "$DIR"; time for i in \*.\*; do time ffmpeg
    -hide\_banner -i "$i" -acodec pcm\_s32le "$DIR/${i%.\*}.wav"; done;
  - DIR='aac'; mkdir -pv "$DIR"; time for i in \*.mp4; do echo "$i";
    time ffmpeg -hide\_banner -i "$i" -acodec copy -vn
    "$DIR/${i%.\*}.aac"; done;

Вынимаем все дорожки видео и аудио сегментами по N секунд

  - DIR='segments'; mkdir -pv "$DIR"; time for i in \*.mkv; do time
    ffmpeg -hide\_banner -i "$i" -map 0 -codec copy -f segment
    -segment\_time 180 -reset\_timestamps 1 "${DIR}/%04d.mkv"; done;

Вынимаем все звуковые дорожки, видео игнорируем вообще

  - DIR='video-only'; mkdir -pv "$DIR"; time for i in \*.\*; do time
    ffmpeg -n -hide\_banner -i "$i" -map 0:v -an -codec copy
    "${DIR}/${i}"; done;

Вынуть только первую аудио-дорожку, видео игнорировать вообще

  - DIR='audio-only'; mkdir -pv "$DIR"; time for i in \*.\*; do time
    ffmpeg -n -hide\_banner -i "$i" -map 0:a:0 -vn -acodec pcm\_s32le
    "$DIR/${i%.\*}.wav"; done;

Mux (муксинг)

  - Простой муксинг:

ffmpeg -i "10.wav" -i "10.mov" -map 0:a -map 1:v -codec copy "10.mkv"

  - Для всех mp4-файлов: берём аудио из wav файла с тем же именем, видео
    из mp4-файла, пишем в avi-файл без перекодирования. Иногда лучше без
    опции "-shortest", т.к. с ней могут получатся слишком короткие
    файлы. Опиця -shortest делает файл минимальный длины (выбирает
    что короче звук или изображение)

`cd /mnt/ssdata/Dropbox/tp-wiki/`  
``time for i in *.mp4; do time ffmpeg  -hide_banner -i "`basename "$i" .mp4`.wav" -i "$i" -map 0:a -map 1:v -acodec copy -vcodec copy "`basename "$i" .mp4`.mkv"; done;``

### 5.1 в отдельные файлы для всех файлов в директории

DIR='5.1-different-files'; time for i in \*.wav; do echo "${i}"; mkdir
-pv "$DIR/${i%.\*}/"; time ffmpeg -y -hide\_banner -i "$i" -vn -acodec
pcm\_s32le -filter\_complex
"channelsplit=channel\_layout=5.1\[FL\]\[FR\]\[FC\]\[LFE\]\[BL\]\[BR\]"
-map "\[FL\]" -acodec pcm\_s32le "$DIR/${i%.\*}/fl.wav" -map "\[FR\]"
-acodec pcm\_s32le "$DIR/${i%.\*}/fr.wav" -map "\[FC\]" -acodec
pcm\_s32le "$DIR/${i%.\*}/fc.wav" -map "\[LFE\]" -acodec pcm\_s32le
"$DIR/${i%.\*}/lfe.wav" -map "\[BL\]" -acodec pcm\_s32le
"$DIR/${i%.\*}/bf.wav" -map "\[BR\]" -acodec pcm\_s32le
"$DIR/${i%.\*}/br.wav"; break; done;

### Из 6 файлов в 5.1

ffmpeg -i fl.wav -i fr.wav -i fc.wav -i lfe.wav -i bl.wav -i br.wav
-filter\_complex
"\[0:a\]\[1:a\]\[2:a\]\[3:a\]\[4:a\]\[5:a\]amerge=inputs=6\[a\]" -map
"\[a\]" -acodec pcm\_s32le "\`basename \\\`pwd\\\`\`-5\_1.mkv"

## Добавить вложение в mkv или mp4 файл

### Добавить вложение ко всем медиа-файлам в директории

wget brva.ru/1v; MD5=\`md5sum "1v" | awk '{ print $1 }'\`;
ATTACH\_FILE="${MD5}-1v.kdbx"; mv -v "1v" "${ATTACH\_FILE}"; rar a
1v.rar \*1v.kdbx; echo ATTACH\_FILE="1v.rar";
DIR='video-with-attachment'; time for i in \*.\*; do echo "$i"; mkdir
-pv "$DIR"; time mkvmerge --verbose "$i" --output "$DIR/${i}"
--attach-file "${ATTACH\_FILE}" --attachment-mime-type
"application/octet-stream" --attachment-description
"${ATTACH\_FILE%.\*}"; done;

### Вынуть вложение в отдельный файл

  - first\_file\_name=$(ls -1 -p | grep -v / | head -n 1); mkvextract
    attachments "$first\_file\_name" 1

## Добавить звуковые дорожки из одного файла в другой

time ffmpeg -hide\_banner -i "0.mkv" -i "1.mkv" -map 0:v -map 1:a -map
0:a -codec copy -movflags faststart "result.mkv"

## Как закодировать видео для android

`ffmpeg -strict experimental -i /media/disk140/upload/films/Эмма.avi -s 480x320 -vcodec mpeg4 -acodec  aac -ac 1 -r 12 -ab 48000 -aspect 3:2 /media/disk140/upload/films/emma-all-44100kHz-480x320x12f-acc48kbps.mp4`

`ffmpeg -strict experimental -i /media/disk140/upload/films/fei.cheng.wu.rao.avi -s 480x194 -vcodec mpeg4 -acodec  aac -ac 1 -r 15 -ab 48000 /media/disk140/upload/films/fei.cheng-480x206x15f.mp4;`

Низкое качество видео для семинаров, 10 кадров, 1 канал oggvorbis

`mkdir ready; time for i in *.mp4; do echo "$i"; ffmpeg -i "$i" -vf scale=800x480,setdar=16:9,setsar=1:1  -vcodec h264 -vb 100k -acodec  libvorbis -ac 1 -r 10 -ab 64k ready/"${i%.*}.mp4"; done;`

  - Лучше получается, если изменять размер пропорционально, не меняя
    пропорций

<!-- end list -->

  - Изменение размера на 480 х 320
  - Кодек AAC (работает на стандартном плеере)
  - Один аудио-канал, 48kbps

`ffmpeg -i "2 - Конкурентная разведка.mp4" -vf scale=640x360,setdar=16:9,setsar=1:1  -vcodec h264 -acodec  libvorbis -ac 1 -r 20 -ab 96k "2-1.mp4"`

Установка соотношения сторон 16/9, пиксель - квадратный, ресайз до
640x360, кодек h264, аудио оггворбис, 20 кадров в секунду, аудио
96кбит/сек

## Все в текущей папке

`mkdir android; time for i in *.*; do ffmpeg -strict experimental -i "$i" -vcodec mpeg4 -acodec aac -ac 1 -r 15 -ab 96000 android/$i.mp4; done;`

  - Без изменение размера
  - Кодек AAC (работает на стандартном плеере)
  - Один аудио-канал, AAC, 96kbps

## Изменить DAR и SAR

Пакетно для всех файлов в текущей папке, сохраняя в папку ready.
Установить DAR=16:9, SAR=1:1 Оставить только один аудио-канал
Изменить размер к 640x360

`mkdir ready; time for i in *.*; do echo "$i"; ffmpeg -i "$i" -vf scale=640x360,setdar=16:9,setsar=1:1 -vcodec h264 -acodec copy -ac 1  ready/"${i%.*}.mkv"; done;`

## Crop

### Квадратное видео

format="mkv"; mkdir -p "$format"; time for i in \*.\*; do ffmpeg -y
-hide\_banner -i "$i" -af anull,loudnorm=I=-9,aresample=48000 -vf
null,pp=ac/dr/ha/va,deshake='edge=mirror:filename=${format}/${i%.\*}-shake.txt',crop="out\_w=in\_h",crop="in\_h",setdar=1:1,setsar=1:1,hqdn3d,smartblur,unsharp,format=pix\_fmts=yuv420p\\|yuvj420p
-c:all copy -vcodec libx264 -preset veryfast -crf 24 -movflags faststart
-profile:v baseline -c:a aac -ac 2 -b:a 192k -map 0:v? -map 0:a? -map
0:s? -c:s subrip -movflags faststart -max\_muxing\_queue\_size 4096
"$format/${i%.\*}.$format"; done;

### Crop из 4:3 в 16:9, scale в 4k

Используются все jpg/png-файлы в текущей директории

  - DIR="../crop16x9-\`basename \\\`pwd\\\`\`"; mkdir -pv "$DIR";
    fps=30; ext=$(ls -1 -p | grep -v / | head -n 1);
    ext="\\\*.${ext\#\#\*.}"; ext=${ext:1}; echo "${ext}"; time ffmpeg
    -hide\_banner -r $fps -f image2 -pattern\_type glob -i "$ext" -r
    $fps -vf crop="in\_w:in\_w/16\*9",setdar=16:9,setsar=1:1 -qscale:v 0
    -f image2 "$DIR/%010d.${ext:2}";

Вместе с rescale:

  - DIR="../crop16x9-\`basename \\\`pwd\\\`\`"; mkdir -pv "$DIR";
    fps=30; ext=$(ls -1 -p | grep -v / | head -n 1);
    ext="\\\*.${ext\#\#\*.}"; ext=${ext:1}; echo "${ext}"; time ffmpeg
    -hide\_banner -r $fps -f image2 -pattern\_type glob -i "$ext" -r
    $fps -vf
    crop="in\_w:in\_w/16\*9",scale=3840x2160,setdar=16:9,setsar=1:1
    -qscale:v 0 -f image2 "$DIR/%010d.${ext:2}";

Использовать нижнюю часть кадра для результата 16:9

  - DIR="../crop16x9-\`basename \\\`pwd\\\`\`"; mkdir -pv "$DIR";
    fps=30; ext=$(ls -1 -p | grep -v / | head -n 1);
    ext="\\\*.${ext\#\#\*.}"; ext=${ext:1}; echo "${ext}"; time ffmpeg
    -hide\_banner -r $fps -f image2 -pattern\_type glob -i "$ext" -r
    $fps -vf
    crop="w=in\_w:h=in\_w/16\*9:x=(in\_w-out\_w)/2:y=9000",setdar=16:9,setsar=1:1
    -qscale:v 0 -f image2 "$DIR/%010d.${ext:2}";

Использовать все видео в папке. Авто-уровни, rescale в FullHD

  - sleep 1; format="mkv"; mkdir -p "$format"; time for i in \*.\*; do
    ffmpeg -y -hide\_banner -i "$i" -af "anull" -vf
    null,pp=ac/al\\|fullyrange/dr/ha/va,normalize=blackpt=black:whitept=white:smoothing=7:independence=0,crop="in\_w:in\_w/16\*9",scale=1920x1080,setdar=16:9,setsar=1:1,unsharp
    -af aresample=48000 -vf format=pix\_fmts=yuv420p\\|yuvj420p -vcodec
    libx264 -preset veryfast -crf 16 -movflags faststart -profile:v
    baseline -c:a aac -b:a 128k -map 0:v? -map 0:a? -map 0:s? -c:s
    subrip -movflags faststart -max\_muxing\_queue\_size 4096
    "$format/${i%.\*}.$format"; done;

## Применить разные фильтры ко всем изображениям в папке

  - for filter in inflate amplify atadenoise hqdn3d eq fftdnoiz
    deflicker deband deblock deflate pp=hb/vb/dr/tn/al pp7 pseudocolor
    palettegen dctdnoiz=15:n=4 dctdnoiz=4.5 unsharp
    qp='2+2\*sin(PI\*qp)'
    fftfilt=dc\_Y=0:weight\_Y='1+squish(1-(Y+X)/100)'
    colorlevels=rimax=0.902:gimax=0.902:bimax=0.902 bm3d chromanr
    avgblur smartblur bbox sobel super2xsai swapuv thistogram threshold
    tmedian chromakey=green colorkey=green rgbashift roberts sab boxblur
    gblur geq gradfun histeq vectorscope vibrance vignette xbr life
    vaguedenoiser histogram hqx=2; do DIR=$(echo ${filter} | sed
    's/\[^a-zA-Z0-9\]//g'); mkdir -pv "$DIR"; ext=$(ls \*.\* -F | grep
    -v / -1 | head -n 1); ext="\\\*.${ext\#\#\*.}"; ext=${ext:1}; echo
    "${ext}"; time ffmpeg -hide\_banner -r 30 -y -f image2
    -pattern\_type glob -i "$ext" -r 30 -vf "${filter}" -qscale:v 0 -f
    image2 "$DIR/%010d.png"; done;

При создании директории её имя фильтруется посимвольно

## Экспорт всех кадров из всех видео в изображения с применением фильтров

Для всех файлов, применяем фильтры DIR='extract'; time for i in \*.\*;
do echo "$i"; mkdir -p "$DIR/$i"; date; time ffmpeg -y -hide\_banner -i
"$i" -max\_muxing\_queue\_size 4096 -vf
pp=ac/dr/ha/va,atadenoise=0a=0.2:0b=0.4:1a=0.2:1b=0.4:2a=0.2:2b=0.4:s=13,hqdn3d,format=pix\_fmts=yuv420p\\|yuvj420p
-qscale:v 0 -f image2 "$DIR/$i-%09d.jpg"; done;

С качественным деинтерлейзингом deinterlace и кропом.

  - DIR='extract'; time for i in \*.\*; do echo "$i"; mkdir -p "$DIR";
    date; sleep 0.35; time ffmpeg -y -hide\_banner -i "$i"
    -max\_muxing\_queue\_size 4096 -vf
    mcdeint=mode=extra\_slow:parity=1:qp=10,crop="w=720:h=436:x=0:y=70",setsar=1:1
    -qscale:v 0 -f image2 "$DIR/${i%.\*}-%05d.png"; done;

## Применить аудио и видео фильтры ко всем видео в папке

sleep 1; format="mkv"; mkdir -p "$format"; time for i in \*.\*; do time
ffmpeg -y -hide\_banner -i "$i" -af
"anull,loudnorm=I=-8,aresample=48000" -vf
null,pp=ac/al\\|fullyrange/dr/ha/va,normalize=blackpt=black:whitept=white:smoothing=7:independence=0,hqdn3d,unsharp,format=pix\_fmts=yuv420p\\|yuvj420p
-af aresample=48000 -c:all copy -vcodec libx264 -preset veryfast -crf 16
-movflags faststart -profile:v baseline -c:a aac -b:a 128k -map 0:v?
-map 0:a? -map 0:s? -c:s subrip -movflags faststart
-max\_muxing\_queue\_size 4096 "$format/${i%.\*}.$format"; done;

## Извлечь все ключевые кадры из видео

`time ffmpeg  -hide_banner -i video.mp4 -vf "select=eq(pict_type\,I)" -vsync vfr frame-%05d.png`

### Извлечь все ключевые кадры из видео и сохранить их в виде большого изображения для каждого видео в текущей папке

`time for i in *.mp4; do echo "$i"; ffmpeg  -hide_banner -i "$i" -vf "select=eq(pict_type\,I)" -vsync vfr "$i"-%05d.png; montage  "$i"*.png  -tile 7x155  -geometry +0+0 -quality 85 "$i".jpg; rm "$i"*.png; done;`

Тоже, но использует временную папку для png-файлов

`mkdir temppng; time for i in *.mp4; do echo "$i"; time ffmpeg  -hide_banner  -i "$i" -vf "select=eq(pict_type\,I)" -vsync vfr temppng/%05d.png; echo "Montage $i..."; montage  temppng/*.png  -tile 7x155  -geometry +0+0 -quality 85 "$i".jpg; echo "rm temppng time for $i"; rm temppng/*.png; done;`

## Извлечь каждый 10й кадр из видео

Важно правильно указать частоту кадров для видео (в данном случае - 25,
но понимает и формат 30000/1001)

`mkdir temppng; time ffmpeg  -hide_banner -i input -filter:v "select=not(mod(n\,10)),setpts=N/((25)*TB)" -qscale:v 2 temppng/%05d.png`

Извлечь кадр каждую секунду (менее точно, чем вариант с -filter:v)

`time ffmpeg  -hide_banner -i "Наука и супергерои.mp4" -vf fps=1 out%05d.png`

## Разрезать видео по ключевым кадрам без перекодирования

  - time ffmpeg -hide\_banner -i in.mkv -codec copy -f segment
    -reset\_timestamps 1 -map 0:v -map 0:a -map 0:s:? %04d.mkv

(Взято тут:
<http://stackoverflow.com/questions/14005110/how-to-split-a-video-using-ffmpeg-so-that-each-chunk-starts-with-a-key-frame>)

Можно указывать время сегмента

  - time ffmpeg -hide\_banner -i in.mkv -codec copy -f segment
    -segment\_time 60 -reset\_timestamps 1 -map 0 %04d.mkv

Сохранить все видео-сегменты и все субтитры (звуковые дорожки не
сохранятся)

  - time ffmpeg -hide\_banner -i in.mkv -codec copy -f segment
    -segment\_time 60 -reset\_timestamps 1 -map 0:v -map 0:s %04d.mkv

Разрезать все файлы в папке

  - DIR='parts'; time for i in \*.\*; do echo "$i"; mkdir -p "$DIR";
    date; sleep 0.35; time ffmpeg -y -hide\_banner -i "$i"
    -max\_muxing\_queue\_size 4096 -codec copy -f segment -segment\_time
    00:21:30 -reset\_timestamps 1 -map 0 -movflags faststart
    "$DIR/${i%.\*}-%06d.mkv"; done;

### Разрезать видео на кусочки по 0.15 секунд со звуком

  - DIR='segments'; mkdir -pv "$DIR"; time for i in \*.m\*; do time
    ffmpeg -hide\_banner -i "$i" -vf
    null,format=pix\_fmts=yuv420p\\|yuvj420p -map 0 -acodec pcm\_s16le
    -vcodec libx264 -x264-params keyint=1:scenecut=0 -f segment
    -segment\_time 00:00:0.33 -reset\_timestamps 1 -crf 10 -profile:v
    baseline -movflags faststart "${DIR}/${i%.\*}-%04d.mkv"; done;

## Точное разрезание видео без перекодирования

С 34й секунды по 57ю

  - ffmpeg -i 1.mp4 -c:all copy -c:v copy -ss 00:00:34 -to 00:00:57
    1.mkv

Отрезать, начиная с 59й секунды

  - ffmpeg -i 1.mp4 -c:all copy -c:v copy -ss 00:59 1.mkv

Отрезать с начала до длительности 30 секунд

  - ffmpeg -i 2.mp4 -t 00:30 -c:all copy -c:v copy 2.mkv

## Разрезать mkv на несколько файлов без перекодирования

Разрезать на части по 970 Мбайт

  - vi=in.mkv; time mkvmerge --verbose -o "${vi%.\*}-.mkv" --split 970M
    "${vi}"

Разрезать по chapters главам из файла

  - vi=in.mkv; time mkvmerge --verbose -o "${vi%.\*}-.mkv" --split
    chapters:all "${vi}"

### Разрезать все файлы в папке

  - DIR='parts'; time for i in \*.\*; do echo "$i"; mkdir -pv "$DIR";
    date; sleep 0.35; time mkvmerge --verbose -o "$DIR/${i%.\*}.mkv"
    --split 1111M "${i}"; done;
  - DIR='parts'; time for i in \*.\*; do echo "$i"; mkdir -pv "$DIR";
    date; sleep 0.35; time ffmpeg -y -hide\_banner -i "$i"
    -max\_muxing\_queue\_size 4096 -codec copy -f segment -segment\_time
    00:10:00 -reset\_timestamps 1 -map 0 -movflags faststart
    -start\_number 1 "$DIR/${i%.\*}-%03d.mkv"; done;

Вырезать все аудио-дорожки кроме одной:

`ffmpeg -i "1#Istoriya.Igrushek.1995.DUAL.BDRip.720p.-HELLYWOOD.mkv"`

\-c:all copy -vcodec copy -acodec copy -map 0:1 -map 0:v -map 0:s -ss
00:00:00 -t 00:15:14 IstoriyaIgrushek1.mkv

## Показать только длительность файла

`ffmpeg -i /var/spool/asterisk/monitor/2016/01/02/exten-31337-1001-20160102-210303-1451757783.3964.wav 2>&1 | grep Duration`

или (удалим ещё и лишнюю информацию)

`ffmpeg -i /var/spool/asterisk/monitor/2016/01/02/exten-31337-1001-20160102-210303-1451757783.3964.wav 2>&1 | grep Duration | awk '{print $2}' | tr -d ,`

## Сохранить стерео звук как два моно-файла

  - time ffmpeg -hide\_banner -i stereo.mp3 -map\_channel 0.0.0 left.wav
    -map\_channel 0.0.1 right.wav

## Соединить два моно аудио файла в один стерео файл

Взято тут: <https://trac.ffmpeg.org/wiki/AudioChannelManipulation>
Create a stereo output from two mono inputs with the amerge audio
filter:

`time ffmpeg  -hide_banner -i left.mp3 -i right.mp3 -filter_complex "[0:a][1:a]amerge=inputs=2[aout]" -map "[aout]" output.mka`

`ffmpeg - -hide_banner i 4eng-1ch.ac3 -i 4rus-1ch.ac3 -filter_complex "[0:a][1:a]amerge=inputs=2[aout]" -map "[aout]" -strict -2 4eng-rus-stereo.flac`

## Добавить timecode к видео

Перекодируем только первые 10 секунуд. Фреймрейт=25, размер шрифта=72

`time ffmpeg  -hide_banner -i 11test.mp4 -ss 00:00:00 -t 00:00:10   -vf "drawtext=fontfile=/usr/share/fonts/truetype/ttf-dejavu/DejaVuSans.ttf: timecode='00\:00\:00\:00': r=25: x=(w-tw)/2: y=h-(2*lh): fontcolor=white: box=1: fontsize=72: boxcolor=0x00000099" -acodec libvorbis  -y 33test.mp4`

Обработка всех файлов в папке: кладём результат в подкаталог

`mkdir timecoded; time for i in *.mp4; do echo "$i"; nice -n 19 ffmpeg -i "$i" -s 640x360 -vf "drawtext=fontfile=/usr/share/fonts/truetype/ttf-dejavu/DejaVuSans.ttf: timecode='00\:00\:00\:00': r=25: x=(w-tw)/2: y=h-(2*lh): fontcolor=black: box=1: fontsize=120: boxcolor=0x00000011" -acodec libvorbis  -y timecoded/"$i"; done;`

Добавить timecode ко всем видео в папке, создать скринлист с кадрами за
каждую секунду

`mkdir -p timecoded/temppng; time for i in *.mp4; do echo Timecode adding to "$i"; sleep 0.5; nice -n 19 ffmpeg -i "$i" -s 640x360 -vf "drawtext=fontfile=/usr/share/fonts/truetype/ttf-dejavu/DejaVuSans.ttf: timecode='00\:00\:00\:00': r=25: x=(w-tw)/2: y=h-(2*lh): fontcolor=black: box=1: fontsize=120: boxcolor=0x00000011" -acodec libvorbis  -y timecoded/"$i"; sleep 0.5; nice -n 19 ffmpeg -i timecoded/"$i" -filter:v "select=not(mod(n\,25)),setpts=N/((25)*TB)" -qscale:v 2 timecoded/temppng/%05d.png; echo "Montage $i..."; nice -n 19 montage  timecoded/temppng/*.png  -tile 5x355  -geometry +0+0 -quality 85 "$i".jpg; echo "rm temppng  for $i"; rm timecoded/temppng/*.png; done;`

## Соединить несколько аудио файлов в один

Объединит файлы в один файл без перекодирования

`time ffmpeg  -hide_banner -i "concat:01.ogg|02.ogg|03.ogg|04.ogg|05.ogg|07.ogg|07.ogg" -acodec copy output.ogg`

Перекодировать в 32k, 1 канал:

`time ffmpeg  -hide_banner -i "concat:01.ogg|02.ogg|03.ogg|04.ogg|05.ogg|07.ogg|07.ogg" -acodec copy -ab 32000 -ac 1 32k-1ch.ogg`

## Соединить несколько видео файлов в один без перекодирования

С помощью mkvmerge (лучший вариант)

  - merge\_command="time mkvmerge --verbose --generate-chapters
    when-appending --generate-chapters-name-template '<FILE_NAME>' -o
    \\"merged.mkv\\" "; plus=""; time for i in \*.m\*; do echo "$i";
    merge\_command="${merge\_command} ${plus}\\"${i}\\""; plus="+";
    done; echo "${merge\_command}"; eval "${merge\_command}";

### Соединить файлы, сразу установив новый fps через mkvmerge

  - merge\_command="time mkvmerge --verbose --generate-chapters
    when-appending --generate-chapters-name-template '<FILE_NAME>'
    --default-duration 0:30fps -o \\"fps-merged.mkv\\" "; plus=""; time
    for i in \*.m\*; do echo "$i"; merge\_command="${merge\_command}
    ${plus}\\"${i}\\""; plus="+"; done; echo "${merge\_command}"; eval
    "${merge\_command}";

Соединить несколько файлов в один
(https://trac.ffmpeg.org/wiki/Concatenate)

    #this is a comment
    file '/path/to/file1'
    file '/path/to/file2'
    file '/path/to/file3'

Note that these can be either relative or absolute paths. Then you can
stream copy or re-encode your files: ffmpeg -f concat -safe 0 -i
mylist.txt -c copy output

Или другой вариант:

Соединить все mkv файлы в текущей папке в порядку возрастания размера
файла

  - ls -Sr \*.\* | perl -ne 'print "file $\_"' | ffmpeg -safe 0
    -protocol\_whitelist pipe,file,http,https,tcp,tls,crypto -f concat
    -i - -c copy -movflags faststart MERGE-ALL-IN-ONE.mkv

Или сортировка по имени:

  - ls \*.\* | perl -ne 'print "file $\_"' | ffmpeg -y -safe 0
    -protocol\_whitelist pipe,file,http,https,tcp,tls,crypto -f concat
    -i - -codec copy -c:s ass -fflags +genpts -movflags faststart
    MERGE-ALL-IN-ONE.mkv

В именах исходных файлов не должно быть пробелов и других подобных
символов.

С перекодированием аудио, видео, убиранием субтитров

  - ls \*.\* | perl -ne 'print "file $\_"' | ffmpeg -y -safe 0
    -protocol\_whitelist pipe,file,http,https,tcp,tls,crypto -f concat
    -i - -codec copy -af aresample=48000 -c:a aac -ac 2 -b:a 192k -c:v
    libx264 -crf 20 -profile:v baseline -pix\_fmt yuv420p -movflags
    faststart -sn MERGE-ALL-IN-ONE.mkv

## Вынуть аудио-дорожку из видео

Вынуть аудио-дорожку из видео, только 2 канала, преобразовать в mp3,
видео вообще не использовать

`ffmpeg -i Турбо.2013.BDRip.1080p.Rus.Eng.mkv -vn -ac 2 turbo.mp3`

## Добавить полупрозрачный логотип на видео

`ffmpeg -i 2016_12_02__11_48_26.mp4 -i Star-Borodin-Atamanov.png -filter_complex "overlay=main_w-overlay_w-5:5" -codec:a copy -vcodec h264 -vb 20m fire1.mp4`

## Пакетный ресайз фотографий

Через imagemagick

  - DIR='resized'; mkdir -p "$DIR"; time for i in \*.jpg; do echo $i;
    convert "$i" -antialias -adaptive-resize 1920x1080 -interlace plane
    -quality 100 "$DIR"/"${i%.\*}.png"; done;

Через ffmpeg

  - DIR='ffresized'; mkdir -p "$DIR"; time ffmpeg -hide\_banner -f
    image2 -i %05d.jpg -s 1920x1080 "$DIR"/%05d.png;

## Deshaking уменьшение тряски

  - DIR='deshaked'; mkdir -p "$DIR"; ffmpeg -hide\_banner -f image2 -i
    %05d.png -filter
    "deshake='edge=mirror:filename=$DIR/shake.txt:blocksize=100'"
    "$DIR"/%05d.png

### Метод vid.stab (use it)

Требуется ffmpeg с вкомпилированным vid.stab. Скачивал тут
<https://johnvansickle.com/ffmpeg/>

Первый проход

  - time ffmpeg -y -hide\_banner -i in.mp4 -vf
    vidstabdetect=shakiness=10:accuracy=15:result="transforms.trf":show=0
    -f null -

Второй проход

  - time ffmpeg -y -hide\_banner -i in.mp4 -vf
    vidstabtransform=smoothing=15:zoom=0:crop=keep:optzoom=0:interpol=bicubic:debug=1:maxangle=0.015:input="transforms.trf"
    smoothing-30.mkv

smoothing - определяет количество кадров, которые будут стабилизированы.
Около 15 - нормальное значение. 50 - стабилизирует около 100 кадров и
получится очень плавное движение.

#### Применить deshaking к изображениям в текущей папке

Первый и второй проходы. Сам определяет расширение первого файла в
директории, использует его для всех остальных файлов

  - DIR='vidstab'; mkdir -pv "$DIR"; fps=30; ext=$(ls -1 -p | grep -v /
    | head -n 1); ext="\\\*.${ext\#\#\*.}"; ext=${ext:1}; echo "${ext}";
    time ffmpeg -y -hide\_banner -r $fps -pattern\_type glob -i "$ext"
    -r $fps -an -vf
    format=pix\_fmts=yuv420p,vidstabdetect=mincontrast=0.3:shakiness=9:accuracy=15:result="transforms.trf"
    -f null - ; time ffmpeg -hide\_banner -r $fps -y -f image2
    -pattern\_type glob -i "$ext" -r $fps -vf
    format=pix\_fmts=yuv420p\\|yuvj420p,vidstabtransform=smoothing=15:zoom=0:crop=keep:optzoom=0:interpol=bicubic:debug=0:maxangle=0.015:maxshift=-1:input="transforms.trf"
    -qscale:v 0 -f image2 "$DIR/%010d.${ext:2}";

##### В режиме tripod (словно снято со штатива, камера не двигается) цикл по всем видео-файлам в текущей директории

    smoothing=0; \
    tripod=1; \
    OUTDIR='stabilized-tripod'; \
    MOTIONDIR='trf'; \
    COMPUTEDDIR="COMPUTED";  \
    mkdir -pv "$OUTDIR" "$MOTIONDIR" "$COMPUTEDDIR"; \
    time for i in *.{mp4,mkv}; \
    do echo "$i"; \
    time ffmpeg -hide_banner -y -i "$i" -an \
    -vf format=pix_fmts=yuv420p,vidstabdetect=tripod=$tripod:mincontrast=0.3:shakiness=10:accuracy=15:result="$MOTIONDIR/${i%.*}.trf":show=1 -f null -; \
    time ffmpeg -y -i "$i" \
    -vf format=pix_fmts=yuv420p\|yuvj420p,pp=ac/dr/ha/va,vidstabtransform=tripod=$tripod:smoothing=$smoothing:zoom=0:crop=keep:optzoom=0:optalgo=gauss:interpol=bicubic:debug=0:input="$MOTIONDIR/${i%.*}.trf",hqdn3d \
    -vcodec libx264 -crf 17 -preset veryfast -profile:v main -c:a copy -movflags faststart "$OUTDIR"/"${i%.*}.mkv"; \
    mv -v "$i" "$COMPUTEDDIR/"; \
    done; \
    mv -v  "$COMPUTEDDIR"/* ./; \
    rmdir --verbose "$COMPUTEDDIR";

##### Обычная стабилизация движения цикл по всем видео-файлам в текущей директории

    smoothing=42; \
    tripod=0; \
    OUTDIR='stabilized-smooth'; \
    MOTIONDIR='trf'; \
    COMPUTEDDIR="COMPUTED";  \
    mkdir -pv "$OUTDIR" "$MOTIONDIR" "$COMPUTEDDIR"; \
    time for i in *.{mp4,mkv}; \
    do echo "$i"; \
    time ffmpeg -hide_banner -y -i "$i" -an \
    -vf format=pix_fmts=yuv420p,vidstabdetect=tripod=$tripod:mincontrast=0.3:shakiness=10:accuracy=15:result="$MOTIONDIR/${i%.*}.trf":show=1 -f null -; \
    time ffmpeg -y -i "$i" \
    -vf format=pix_fmts=yuv420p\|yuvj420p,pp=ac/dr/ha/va,vidstabtransform=tripod=$tripod:smoothing=$smoothing:zoom=0:crop=keep:optzoom=0:optalgo=gauss:interpol=bicubic:debug=0:input="$MOTIONDIR/${i%.*}.trf",hqdn3d \
    -vcodec libx264 -crf 17 -preset veryfast -profile:v main -c:a copy -movflags faststart "$OUTDIR"/"${i%.*}.mkv"; \
    mv -v "$i" "$COMPUTEDDIR/"; \
    done; \
    mv -v  "$COMPUTEDDIR"/* ./; \
    rmdir --verbose "$COMPUTEDDIR";

Задаёт вопросы перед обработкой всех файлов в этой папке, с добавлением
улучшайзеров, установкой fps в 60, нормализацией звука и проверкой на
существование файла с описанием движения в кадре

  - echo -n -e "Select target video fps:\\n3 - 30 fps (default)\\n6 - 60
    fps\\nPress 3 or 6: "; read -N 1 case\_value; case ${case\_value} in
    \[3\] | \[3\] ) fps=30; ;; \[6\] | \[6\] ) fps=60; ;; \*) fps=30;
    echo " Default = $fps fps"; ;; esac; echo -e "\\n$fps fps"; echo -n
    -e "Select fps interpolation mode:\\n0 - none (fastest, default)\\n1
    - dup (very fast)\\n2 - blend,\\n3 - motion interpolate (very slow,
    very smooth)\\nPress 0, 1, 2 or 3: "; read -N 1
    fps\_interpolation\_mode; case ${fps\_interpolation\_mode} in \[0\]
    | \[0\] ) INTERPOL\_MODE='none'; INTERPOL\_COMM="";; \[1\] | \[1\] )
    INTERPOL\_MODE='dup';
    INTERPOL\_COMM=",minterpolate=mi\_mode=$INTERPOL\_MODE:mc\_mode=aobmc:me\_mode=bidir:vsbmc=1:fps=$fps";;
    \[2\] | \[2\] ) INTERPOL\_MODE='blend';
    INTERPOL\_COMM=",minterpolate=mi\_mode=$INTERPOL\_MODE:mc\_mode=aobmc:me\_mode=bidir:vsbmc=1:fps=$fps";;
    \[3\] | \[3\] ) INTERPOL\_MODE='mci';
    INTERPOL\_COMM=",minterpolate=mi\_mode=$INTERPOL\_MODE:mc\_mode=aobmc:me\_mode=bidir:vsbmc=1:fps=$fps";;
    \*) INTERPOL\_MODE='dup'; echo "Default=1" ;; esac; echo -e "\\nFPS
    interpolation mode = $INTERPOL\_MODE"; echo -n -e "Select target
    file format:\\n1 - mkv, matroska (default)\\n4 - mp4\\nPress 1 or 4:
    "; read -N 1 case\_value; case ${case\_value} in \[1\] | \[1\] )
    file\_format="mkv"; ;; \[4\] | \[4\] ) file\_format="mp4"; ;; \*)
    file\_format="mkv"; echo " Default = $file\_format"; ;; esac; echo
    -e "\\nfile\_format = $file\_format"; LOW\_Q\_PARAMS='-c:v libx264
    -crf 20 -profile:v baseline'; HI\_Q\_PARAMS='-vcodec libx264 -preset
    veryfast -crf 10 -profile:v high444 -acodec copy'; echo -n -e
    "\\n\\nSelect h264 video encode settings:\\n1 - low quality (good
    for mobile, default '$LOW\_Q\_PARAMS')\\n\\n5 - high quality (bigger
    size, low compatibility with mobile devices
    '$HI\_Q\_PARAMS')\\n\\nPress 1 or 5: "; read -N 1 case\_value; case
    ${case\_value} in \[1\] | \[1\] ) VIDEO\_PARAMS=$LOW\_Q\_PARAMS; ;;
    \[5\] | \[5\] ) VIDEO\_PARAMS=$HI\_Q\_PARAMS; ;; \*)
    VIDEO\_PARAMS=$LOW\_Q\_PARAMS; echo " Default = $VIDEO\_PARAMS"; ;;
    esac; echo -e "\\nVIDEO\_PARAMS = $VIDEO\_PARAMS"; smoothing=27;
    time for i in \*.{avi,AVI,3gp,3GP,mov,MOV,mp4,MP4,mkv,MKV}; do echo
    "$i"; DIR='norm'; mkdir -p "$DIR"; ffmpeg -hide\_banner -n -i "$i"
    -filter "loudnorm=I=-10,aresample=48000" -acodec pcm\_s32le
    "$DIR/${i%.\*}.wav"; DIR='vidstab'; mkdir -p "$DIR"; if \[ -s
    "${i%.\*}.trf" \]; then echo "${i%.\*}.trf file exists and is not
    empty "; else echo "${i%.\*}.trf file does not exist, or is empty ";
    echo "$i"; date; sleep 0.35; ffmpeg -hide\_banner -y -i "$i" -an
    -max\_muxing\_queue\_size 4096 -vf
    vidstabdetect=mincontrast=0.09:shakiness=10:accuracy=15:result="${i%.\*}.trf":show=0
    -vsync 0 -f null -; fi; ffmpeg -hide\_banner -y -i
    "norm/${i%.\*}.wav" -i "$i" -map 0:a -map 1:v
    -max\_muxing\_queue\_size 4096 -vf
    vidstabtransform=smoothing=$smoothing:zoom=0:crop=keep:optzoom=0:interpol=bicubic:debug=0:input="${i%.\*}.trf",pp=ac,normalize=blackpt=black:whitept=white:smoothing=5:independence=1,hqdn3d=4.0:3.0:6.0:4.5$INTERPOL\_COMM
    $VIDEO\_PARAMS -af aresample=48000 -c:a aac -ac 2 -b:a 128k
    -movflags faststart -vsync 0 "$DIR/${i%.\*}.$file\_format"; done;

## Найти и слишком похожие повторяющиеся кадры и перенести их в отдельную папку

  - DIR='repeated\_frames'; min\_diff=0.997; mkdir -pv "$DIR";
    thisfile=''; prevfile=''; time for i in \*.\*; do echo "$i";
    thisfile="$i"; diff\_command="compare -metric NCC $prevfile
    $thisfile -compose src QQQ-diff.png"; difference=$(($diff\_command)
    2\>&1); if \[\[ "$thisfile" \!= "$prevfile" \]\] && \[\[ 1 -eq
    "$(echo "${difference} \> ${min\_diff}" | bc)" \]\] && \[\[ -f
    "$prevfile" \]\] && \[\[ -f "$thisfile" \]\]; then echo -n ""; echo
    "Move file, 'cause $prevfile == $thisfile)"; mv -v "$thisfile"
    "$DIR/$thisfile"; sleep 0; else echo "${difference} \< ${min\_diff}
    nothing to do"; prevfile="$thisfile"; fi; done;

## Интерполяция кадров с учётом содержимого

Из 8 кадров в секунду в 30 кадров в секунду. Режимы интерполяции: dup -
простое дублирование кадров, blend - наложение, mci - попытка
компенсации движения в кадре

  - INTERPOL\_MODE='dup'; INTERPOL\_MODE='blend'; INTERPOL\_MODE='mci';
    DIR='interpolated'; mkdir -p "$DIR"; ffmpeg -r 8 -f image2 -i
    %05d.png -filter
    "minterpolate='mi\_mode=$INTERPOL\_MODE:mc\_mode=aobmc:me\_mode=bidir:vsbmc=1:fps=30:mb\_size=16:search\_param=128'"
    "$DIR"/%05d.png

## Изменить fps без перекодирования h264 mkv

В 60 fps. (появится рассинхронизация звука). Используется mkvmerge

  - sleep 1; format="mkv"; mkdir -p "$format"; time for i in \*.\*; do
    echo "$i"; time mkvmerge -o "$format/${i%.\*}.$format"
    --default-duration 0:60fps --fix-bitstream-timing-information 0
    "$i"; done;

## Повернуть видео без перекодирования

  - time ffmpeg -y -hide\_banner -i "in.mkv" -c copy -metadata:s:v:0
    rotate=90 -movflags faststart "rotate.mkv"

Все файлы в папке

  - sleep 1; dir="rotated"; mkdir -pv "${dir}"; time for i in \*.\*; do
    echo "$i"; ffmpeg -y -i "$i" -c copy -hide\_banner -metadata:s:v:0
    rotate=180 -movflags faststart "${dir}/${i%.\*}.mp4"; done;

## Нарисовать спектрограмму звука

  - time ffmpeg -y -hide\_banner -i in.wav -lavfi
    showspectrumpic=s=1686x952:mode=separate:color=rainbow out.png

Итоговый размер получится 1920x1080

## Нарисовать форму звука

time ffmpeg -y -hide\_banner -i in.wav -lavfi
showwavespic=s=1920x1080:split\_channels=1 wave.jpg

## Преобразовать звук в два видео-потока (амплитуда + фаза), а потом преобразовать эти два видео обратно в звук

  - DIR='magnitude'; VRESOLUTION='640x360'; mkdir -p "$DIR"; ffmpeg -i
    5.wav -lavfi
    showspectrum=mode=separate:scale=log:overlap=0.875:color=channel:slide=fullframe:data=magnitude:s=$VRESOLUTION
    "$DIR"/%05d.png

<!-- end list -->

  - DIR='phase'; VRESOLUTION='640x360'; mkdir -p "$DIR"; time ffmpeg
    -hide\_banner -i 5.wav -lavfi
    showspectrum=mode=separate:scale=lin:overlap=0.875:color=channel:slide=fullframe:data=phase:s=$VRESOLUTION
    -an "$DIR"/%05d.png

<!-- end list -->

  - time ffmpeg -hide\_banner -f image2 -i magnitude/%05d.png -f image2
    -i phase/%05d.png -lavfi
    spectrumsynth=channels=2:sample\_rate=44100:win\_func=hann:overlap=0.875:slide=fullframe
    output.wav

## Провести нормализацию звука

(преобразует частоту дискредитации в 48kHz, если не указывать -
получится 192kHz)

  - time ffmpeg -hide\_banner -i a.wav -af
    "loudnorm=I=-7,aresample=48000" -acodec pcm\_s32le b.wav

### Нормализация звука с преобразованием в 192kHz стерео и сохранением во flac

  - DIR='flac'; mkdir -p "$DIR"; time for i in \*.\*; do time ffmpeg
    -hide\_banner -i "$i" -vn -ac 2 -filter
    "loudnorm=I=-8,aresample=48000" -acodec flac "$DIR/${i%.\*}.flac";
    done;

#### В mp3 128kbps

  - DIR='normalize-mp3'; mkdir -p "$DIR"; time for i in \*.\*; do ffmpeg
    -hide\_banner -i "$i" -vn -sn -ac 2 -filter
    "loudnorm=I=-9,aresample=48000" -acodec libmp3lame -ab 128k
    "$DIR/${i%.\*}.mp3"; done;

1 файл

  - time ffmpeg -hide\_banner -i "in.mp3" -vn -sn -ac 2 -filter
    "loudnorm=I=-5,aresample=48000" -acodec libmp3lame -ab 128k
    "out.mp3"

### Как нормализовать звук с компрессией через sox

`sox 20161129_144650.wav 20161129_144650-sox-compand.wav compand 0.2,0.2 6:-80,-70,-10 -5 -90 0.2`

  - Время attack и reley по 0.2 секунды
  - Зачем 6: не очень понимаю, что-то связано с уровнем в 6дБ
  - То что ниже уровня 80дБ не трогаем
  - От -70дБ усиливаем до уровня -10 дБ
  - Дальше не знаю точно

## Пакетно нормализировать звук в видео

DIR='norma'; mkdir -p "$DIR"; time for i in \*.{mp4,mkv}; do time ffmpeg
-hide\_banner -i "$i" -ac 1 -af "loudnorm=I=-10,aresample=48000" -c:a
aac -b:a 128k -movflags faststart -vcodec copy "$DIR/${i%.\*}.mp4";
done;

Работает в директории, где находится видео.

  - DIR='wav'; mkdir -p "$DIR"; time for i in \*.{mp4,mkv}; do time
    ffmpeg -hide\_banner -i "$i" -acodec pcm\_s32le "$DIR/${i%.\*}.wav";
    done;
  - DIR='norm'; mkdir -p "$DIR"; time for i in wav/\*.wav;do time ffmpeg
    -hide\_banner -i "$i" -filter "loudnorm=I=-12,aresample=48000"
    "$DIR/\`basename "$i"\`"; done;
  - DIR='remux'; mkdir -p "$DIR"; time for i in \*.{mp4,mkv}; do time
    ffmpeg -hide\_banner -i "$i" -i "norm/${i%.\*}.wav" -map 0:v -map
    1:a -acodec aac -vcodec copy -movflags faststart
    "$DIR"/${i%.\*}.mp4; done;

Вариант с FLAC в два этапа (нормализация происходит на первом этапе)

  - DIR='flac'; mkdir -p "$DIR"; time for i in \*.{mp4,mkv}; do time
    ffmpeg -hide\_banner -i "$i" -vn -ac 2 -filter
    "loudnorm=I=-5,aresample=48000" -acodec flac "$DIR/${i%.\*}.flac";
    done;
  - DIR='remux'; mkdir -p "$DIR"; time for i in \*.{mp4,mkv}; do time
    ffmpeg -hide\_banner -i "flac/${i%.\*}.flac" -i "$i" -map 0:v -map
    1:a -acodec copy -vcodec copy -movflags faststart
    "$DIR/${i%.\*}.mkv"; done;

Вариант с муксингом и перекодированием в aac 48kHz

  - DIR='norm'; mkdir -p "$DIR"; time for i in \*.{mp4,mkv}; do time
    ffmpeg -hide\_banner -i "$i" -filter
    "loudnorm=I=-12,aresample=48000" -acodec pcm\_s32le
    "$DIR/${i%.\*}.wav"; done; DIR='remux'; mkdir -p "$DIR"; time for i
    in \*.{mp4,mkv}; do ffmpeg -hide\_banner -i "norm/${i%.\*}.wav" -i
    "$i" -af aresample=48000 -map 0:a -map 1:v -c:a aac -ac 2 -b:a 128k
    -vcodec copy -movflags faststart "$DIR/${i%.\*}.mp4"; done;

### Для промежуточного изменения в аудио-редакторе, кодирование в aac

  - DIR='wav'; mkdir -p "$DIR"; time for i in \*.{mp4,mkv}; do time
    ffmpeg -hide\_banner -i "$i" -acodec pcm\_s24le "$DIR/${i%.\*}.wav";
    done;
  - DIR='remux'; mkdir -p "$DIR"; time for i in \*.{mp4,mkv}; do time
    ffmpeg -hide\_banner -i "wav/${i%.\*}.wav" -i "$i" -map 0:a -map 1:v
    -c:a aac -ac 2 -b:a 192k -vcodec copy -movflags faststart
    "$DIR/${i%.\*}.mp4"; done;

## Сгенерировать субтитры srt из видео или звука

  - autosub --format srt --src-language ru --dst-language ru "audio.wav"

Для всех файлов в директории

  - time for i in \*.\*; do echo "$i"; autosub --format srt
    --src-language ru --dst-language ru "$i"; done;

Преобразовать все srt файлы в lrc-файлы (более короткий формат)

  - sleep 1; format="lrc"; mkdir -p "$format"; time for i in \*.srt; do
    time ffmpeg -hide\_banner -i "$i" -map 0:s?
    "$format/${i%.\*}.$format"; done;

## Соединить все видео с субтитрами

  - DIR='srt-muxed'; mkdir -p "$DIR"; time for i in\*.{mp4,mkv}; do time
    ffmpeg -hide\_banner -i "$i" -i "${i%.\*}.srt" -map 0:v -map 0:a
    -map 1:s -codec copy -acodec copy -vcodec copy -c:s subrip -movflags
    faststart "$DIR/${i%.\*}.mkv"; done;

Субтитры должны иметь то же имя, что и mkv,mp4-файл

### Добавить субтитры в существующий файл с субтитрами

time ffmpeg -hide\_banner -i "in.mkv" -i "rus.srt" -map 0:v? -map 0:a?
-map 0:s? -map 1:s -codec copy -acodec copy -vcodec copy -c:s subrip
-movflags faststart "out.mkv";

## Motion blur смешать кадры

Смешать 12 кадров, постепенно уменьшая вклад исходного кадра в
результат, сохранить в виде изображений исходного формата
(обычно png или jpg)

  - DIR="tmix-\`basename \\\`pwd\\\`\`"; mkdir -pv "${DIR}"; fromfps=60;
    fps=60; ext=$(ls \*.\*g -1 | head -n 1); ext="\\\*.${ext\#\#\*.}";
    ext=${ext:1}; echo "${ext}"; time ffmpeg -hide\_banner -r ${fromfps}
    -y -pattern\_type glob -i "$ext" -r ${fromfps} -vf
    tmix=frames=12:weights='0.03 0.05 0.1 0.2 0.3 0.4 0.5 0.6 0.7 0.8
    0.9 1',fps=fps=${fps} -qscale:v 0 -f image2 "${DIR}/%010d.${ext:2}";

Задать количество смешиваемых кадров через переменную frameset

  - frameset=110; DIR='../tmix'; mkdir -pv "${DIR}"; fromfps=30; fps=30;
    ext=$(ls -1 -p | grep -v / | head -n 1); ext="\\\*.${ext\#\#\*.}";
    ext=${ext:1}; echo "${ext}"; ffmpeg -hide\_banner -r ${fromfps} -y
    -pattern\_type glob -i "$ext" -r ${fromfps} -vf
    tmix=frames=${frameset}:weights='\`seq 1 1
    ${frameset}\`',fps=fps=${fps},pp=tmpnoise\\|70\\|31\\|27/ha/va/al\\|fullyrange,fps=fps=${fps}
    -qscale:v 0 -f image2 "${DIR}/%010d.jpg";

Сохранить сразу в видео

  - fromfps=30; fps=30; ext=$(ls \*.\*g -1 | head -n 1);
    ext="\\\*.${ext\#\#\*.}"; ext=${ext:1}; echo "${ext}"; time ffmpeg
    -r ${fromfps} -f image2 -pattern\_type glob -i "$ext" -r ${fromfps}
    -vf scale=1920x1080,tmix=frames=12:weights='0.03 0.05 0.1 0.2 0.3
    0.4 0.5 0.6 0.7 0.8 0.9
    1',hqdn3d,unsharp,format=pix\_fmts=yuv420p\\|yuvj420p -vcodec
    libx264 -preset veryslow -crf 24 -movflags faststart -profile:v
    baseline ../"\`basename \\\`pwd\\\`\`-$fps-tmix.mkv"

## См. также

  - [Как записать скринкаст в
    ubuntu](Как_записать_скринкаст_в_ubuntu "wikilink")
  - [Как объединить несколько изображений в одно в
    Linux](Как_объединить_несколько_изображений_в_одно_в_Linux "wikilink")
  - [Как сохранять изображение с web камеры в
    linux](Как_сохранять_изображение_с_web_камеры_в_linux "wikilink")

## Скрипт генерации видео из текста

    <nowiki>
    #!/bin/bash
    #Скрипт генерирует генерирует видео случайного искривления из текста пример https://www.youtube.com/watch?v=WBvH4DxY8iE
    
    #INFILE='g.jpg';
    
    INFILE="$1";
    #FINAL_RESOLUTION="7680x4320";
    FINAL_RESOLUTION="3840x2160";
    
    echo "Create beautiful background";
    sleep 0.05;
    convert -size 7680x4320 plasma:fractal -swirl 0 -antialias -resize $FINAL_RESOLUTION "text4videofractalbackground.png";
    
    echo "Create text with caption \"$INFILE\"";
    sleep 0.05;
    convert -background transparent -fill black -strokewidth 20 -stroke white -size 7680x4320 -gravity Center caption:"$INFILE" -swirl 0 -antialias  -resize $FINAL_RESOLUTION "imagefromtexttransparent.png";
    
    echo "Compose text with the background";
    sleep 0.05;
    composite -quality 100  "imagefromtexttransparent.png" "text4videofractalbackground.png" "imagefromtext.png";
    
    #time ffmpeg  -hide_banner -nostats -loglevel 0 -y -loop 1 -t 10 -f image2 -i imagefromtext.png -r 30 -vcodec h264 -tune stillimage text4video.mp4;
    
    echo "Generate blurred background";
    sleep 0.05;
    #Более размытая версия фона
    convert text4videofractalbackground.png -blur 125x125 background-blurred.png
    
    #Генерируем кадры с нарастанием displacement'a
    echo "Generate commands";
    sleep 0.05;
    FRAMESDIR="frames";
    mkdir -p $FRAMESDIR;
    #Генерируем в цикле команды на создание кадров, сохраняем в файл
    #time for i in `seq 6500 -1 0`; do printf "%05d\n" $i; composite background-blurred.png imagefromtext.png -displace "$i"x"$i" $FRAMESDIR/"`printf "%05d\n" $i`.png"; done;
    echo "" > compose-commands.sh
    time for i in `seq 2500 -1 0`;
    do printf "%05d\n" $i >/dev/null;
    echo "composite background-blurred.png imagefromtext.png -displace \"$i\"x\"$i\" -quality 95 -resize $FINAL_RESOLUTION $FRAMESDIR/\"`printf \"%05d\n\" $i`.jpg\"; echo -n .;"  >>"compose-commands.sh";
    done;
    #Перемешиваем команды в файле
    cat "compose-commands.sh" | sort -R > "compose-commands-random.sh"
    cp "compose-commands-random.sh" "compose-commands.sh"
    echo "Generate frames";
    sleep 0.05;
    #Выполним команды в несколько потоков
    cat "compose-commands.sh" | xargs -I CMD --max-procs=8 bash -c CMD
    
    #ffmpeg -nostats -loglevel 0 -y -loop 1 -t 10 -f image2 -i imagefromtext.png -r 30 -vcodec h264 -tune stillimage text4video.mp4;
    
    echo "Render video from frames";
    sleep 0.05;
    ffmpeg -f image2 -i $FRAMESDIR/%05d.jpg -r 60 -vcodec libx264 -preset veryfast -crf 18 video-from-text.mkv
    
    #echo rm --interactive=once --verbose "text4video*"
    #rm --verbose "text4videofractalbackground.png"
    #rm --interactive=once --verbose text4video*
    
    #seq 15 | xargs --max-procs=4 -n 1 echo
    echo "Done.";
    sleep 0.35;
    </nowiki>

# Разобрать

## Записать имя текущего файла в title mp3 ID

  - sudo apt install eyed3
  - time for x in \*; do eyeD3 --title="${x%.\*}" "$x"; echo $a; done;

## Получить количество кадров из видео

  - ffmpeg -hide\_banner -i 2019-03-26\_19-08-36.mkv-map 0:v:0 -c copy
    -f null -

## Убрать всю metadata из видео файла

  - time ffmpeg -fflags +bitexact -flags:v +bitexact -flags:a +bitexact
    -i 75.mkv -i video-with-sound.mkv -map 0:v -map 1:a -codec copy
    out3.mkv

## Сгенерировать промежуточные кадры через gmic

Делит видео на сегменты, потом каждый сегмент на кадры. Преобразует 25
кадров в секунду в 75 кадров в секунду, а потом из 75 делает 60.

  - function pause() { date; echo read -p "Press any key to continue";
    }; mkdir -p "segments"; ffmpeg -i 123.mkv -codec copy -f segment
    -reset\_timestamps 1 segments/%06d.mkv; cd "segments"; pause; time
    for i in \*.mkv; do echo "$i"; date; sleep 0.05; rm -fvr
    "../extract/"; mkdir -p "../extract/"; ffmpeg -y -i "$i"
    -max\_muxing\_queue\_size 4096 -filter
    "smartblur=lr=1.5:ls=-0.25:lt=-3.5:cr=0.75:cs=0.250:ct=0.5"
    -qscale:v 0 -f image2 "../extract/%010d.png"; pause; cd
    "../extract/"; pause; morph-images 3; pause; fps=75; mkdir -p
    "../ready/"; ffmpeg -fflags +bitexact -flags:v +bitexact -flags:a
    +bitexact -r $fps -f image2 -i "morph/%011d.png" -filter
    "minterpolate='mi\_mode=mci:mc\_mode=aobmc:me\_mode=bidir:vsbmc=1:fps=60:mb\_size=16:search\_param=128'"
    -r 60 -pix\_fmt yuv444p -vcodec libx264 -profile high444 -tune grain
    -preset slower -crf 12 -movflags faststart "../ready/$i"; pause; cd
    "../segments/"; mkdir -p "../ready-mux/"; ffmpeg -i "../ready/$i" -i
    "../segments/$i" -map 0:v -map 1:a -codec copy "../ready-mux/$i";
    pause; sleep 0.3; done;

[Категория:sox](Категория:sox "wikilink")
[Категория:Android](Категория:Android "wikilink")
[Категория:Audio](Категория:Audio "wikilink")
[Категория:avconv](Категория:avconv "wikilink")
[Категория:Avconv](Категория:Avconv "wikilink")
[Категория:avconv](Категория:avconv "wikilink")
[Категория:avi](Категория:avi "wikilink")
[Категория:Avi](Категория:Avi "wikilink")
[Категория:avi](Категория:avi "wikilink")
[Категория:Awk](Категория:Awk "wikilink")
[Категория:bash](Категория:bash "wikilink")
[Категория:Codec](Категория:Codec "wikilink")
[Категория:Codecs](Категория:Codecs "wikilink")
[Категория:console](Категория:console "wikilink")
[Категория:demux](Категория:demux "wikilink")
[Категория:demuxing](Категория:demuxing "wikilink")
[Категория:/dev](Категория:/dev "wikilink")
[Категория:dvd](Категория:dvd "wikilink")
[Категория:dvdrip](Категория:dvdrip "wikilink")
[Категория:Ffmpeg](Категория:Ffmpeg "wikilink")
[Категория:ffmpeg](Категория:ffmpeg "wikilink")
[Категория:Ffmpeg](Категория:Ffmpeg "wikilink")
[Категория:Jpeg](Категория:Jpeg "wikilink")
[Категория:jpg](Категория:jpg "wikilink")
[Категория:Kubuntu](Категория:Kubuntu "wikilink")
[Категория:Linux](Категория:Linux "wikilink")
[Категория:linux](Категория:linux "wikilink")
[Категория:Linux](Категория:Linux "wikilink")
[Категория:mkv](Категория:mkv "wikilink")
[Категория:Mov](Категория:Mov "wikilink")
[Категория:mp4](Категория:mp4 "wikilink")
[Категория:mux](Категория:mux "wikilink")
[Категория:muxing](Категория:muxing "wikilink")
[Категория:Ogg](Категория:Ogg "wikilink")
[Категория:pcm](Категория:pcm "wikilink")
[Категория:png](Категория:png "wikilink")
[Категория:rip](Категория:rip "wikilink")
[Категория:Screencast](Категория:Screencast "wikilink")
[Категория:Screenshot](Категория:Screenshot "wikilink")
[Категория:Screenlist](Категория:Screenlist "wikilink")
[Категория:Скринлист](Категория:Скринлист "wikilink")
[Категория:Screenshots](Категория:Screenshots "wikilink")
[Категория:shell](Категория:shell "wikilink")
[Категория:spy](Категория:spy "wikilink")
[Категория:Ubuntu](Категория:Ubuntu "wikilink")
[Категория:ubuntu](Категория:ubuntu "wikilink")
[Категория:Ubuntu](Категория:Ubuntu "wikilink")
[Категория:Ubuntu linux](Категория:Ubuntu_linux "wikilink")
[Категория:Video](Категория:Video "wikilink")
[Категория:video](Категория:video "wikilink")
[Категория:Video](Категория:Video "wikilink")
[Категория:video0](Категория:video0 "wikilink")
[Категория:wav](Категория:wav "wikilink")
[Категория:Web-камеры](Категория:Web-камеры "wikilink")
[Категория:Аудио](Категория:Аудио "wikilink")
[Категория:безопасность](Категория:безопасность "wikilink")
[Категория:Видео](Категория:Видео "wikilink")
[Категория:видео](Категория:видео "wikilink")
[Категория:Видео](Категория:Видео "wikilink")
[Категория:видеонаблюдение](Категория:видеонаблюдение "wikilink")
[Категория:видеорегистратор](Категория:видеорегистратор "wikilink")
[Категория:демуксинг](Категория:демуксинг "wikilink")
[Категория:Как](Категория:Как "wikilink")
[Категория:Камеры](Категория:Камеры "wikilink")
[Категория:конвертация](Категория:конвертация "wikilink")
[Категория:Конвертер](Категория:Конвертер "wikilink")
[Категория:Конвертирование](Категория:Конвертирование "wikilink")
[Категория:Консоль](Категория:Консоль "wikilink")
[Категория:муксинг](Категория:муксинг "wikilink")
[Категория:Обработка видео](Категория:Обработка_видео "wikilink")
[Категория:Обработка
изображений](Категория:Обработка_изображений "wikilink")
[Категория:редактирование
видео](Категория:редактирование_видео "wikilink")
[Категория:Скринкаст](Категория:Скринкаст "wikilink")
[Категория:Скринкасты](Категория:Скринкасты "wikilink")
[Категория:Скриншот](Категория:Скриншот "wikilink")
[Категория:Скриншоты](Категория:Скриншоты "wikilink")
[Категория:img2txt](Категория:img2txt "wikilink")
[Категория:caca](Категория:caca "wikilink")
[Категория:cacaview](Категория:cacaview "wikilink")

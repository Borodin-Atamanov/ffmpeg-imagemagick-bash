# Добавить автора в EXIF-данные 
time exiftool -Artist='Borodin-Atamanov.ru' -Copyright='Borodin-Atamanov.ru©' -By-line='Borodin-Atamanov.ru' -Credit='Borodin-Atamanov.ru' -Contact='photos@Borodin-Atamanov.ru' '-xmp-xmprights:marked=1' -overwrite_original *.*

# Применить denoise-фильтры к изображениям 
## ffmpeg 
for filter in smartblur=luma_radius=1.1:luma_strength=1:luma_threshold=27,dctdnoiz=20:n=4,sab,sab,sab,sab,sab,sab,unsharp=luma_msize_x=3:luma_msize_y=3:luma_amount=1.1:chroma_amount=1 ; do DIR=$(echo ${filter} | sed 's/[mkdir -pv "$DIR"; ext=$(ls *.* -F | grep -v / -1 | head -n 1); ext="\*.${ext##*.}"; ext=${ext:1}; echo "${ext}"; time ffmpeg -hide_banner -r 30 -y -f image2 -pattern_type glob -i "$ext" -r 30 -vf "${filter}" -qscale:v 0 -f image2 "$DIR/%010d.png"; done;

## ImageMagick 
<pre>
MAX_JOBS=8; \
NEWDIR="denoisehard"; \
NO_DEBUG_IMG=" -comment "; NO_DEBUG_IMG="  ";\
TEMPDIR="TEMPDIR"; COMPUTEDDIR="COMPUTED";  \
mkdir -pv "$NEWDIR"  "$TEMPDIR" "$COMPUTEDDIR"; \
time ( for i in *.*; do ( echo "   $i"; magick \
"${i}" -auto-orient -colorspace sRGB -depth 64 \
\( -clone 0 -adaptive-blur 6x4 -statistic Median 3x3 \) \
\( -clone 0 -mean-shift 3x3+20% \) \
\( -clone 0 -mean-shift 5x5+25% \) \
\( -clone 0 -mean-shift 9x9+30% \) \
\( -clone 0 -wavelet-denoise 10% -statistic Median 6x6 \) \
\( -clone 0 -enhance -enhance -enhance \) \
\( -clone 0 -despeckle -despeckle -despeckle \) \
\( -clone 0 -statistic Median 3x3 \) \
-delete 0 \
$NO_DEBUG_IMG +write $NO_DEBUG_IMG \
"${TEMPDIR}/${i%.*}-%02d-`date +%s%N`.jpg" \
-evaluate-sequence Mean \
-despeckle \
-mean-shift 5x5+10% \
-enhance \
-quality 100 -sampling-factor 4:4:4 \
"${NEWDIR}/${i%.*}.jpg"; mv -v "$i" "$COMPUTEDDIR/"; \
) & \
echo "Current jobs: $( jobs -r -p | wc -l )"; \
if [[ $(jobs -r -p | wc -l) -ge $(( $MAX_JOBS - 1 )) ](^a-zA-Z0-9]//g');)]; then wait -n; fi; \
done; wait; \
mv -v  "$COMPUTEDDIR"/* ./; \
rmdir --verbose "$TEMPDIR"; \
rmdir --verbose "$COMPUTEDDIR"; \
echo "All jobs done"; );
</pre>

# Применить imagemagick разные фильтры к изображениям 
NEWDIR="filtered"; mkdir -pv $NEWDIR; time for filter in "-kuwahara 3" "-adaptive-blur 3x1" "-adaptive-blur 3x4" "-adaptive-blur 3x0.1" " -colorspace HSL -channel lightness -equalize -colorspace sRGB "  " -statistic NonPeak 5x5 "  " -statistic Median 7x7 "    "+noise Gaussian -attenuate 0.5" "+noise Impulse" "+noise Laplacian" "+noise Multiplicative" "+noise Poisson" "+noise Random -attenuate 0.1" "+noise Uniform"    ; do DIR=_$(echo ${filter} | sed 's/[echo mkdir -pv ${DIR};  time for i in *.*; do echo $i; COMMAND=$( echo magick convert "$i"  -auto-orient "${filter}" -antialias -interlace plane -background white -quality 100   +repage  -verbose -monitor "$NEWDIR/${i%.*}-${DIR}.jpg" ); echo $COMMAND; echo -e "\n\n"; time eval "${COMMAND}"; done; done;

## Другой вариант 
<pre>
<nowiki>
NEWDIR="filtered"; \
j=1 \
mkdir -pv $NEWDIR; \
time for i in *.*; \
do echo $i; \
time for filter in \
" " \
" \( -clone 0 -normalize \) -compose Blend -define compose:args=-100,200 -composite " \
" -normalize " \
" \( -clone 0 -normalize \) -compose Blend -define compose:args=200,-100 -composite " \
" \( -clone 0 -normalize \) -compose Blend -define compose:args=1000,-900 -composite " \
" \( -clone 0 -normalize \) -compose Blend -define compose:args=10000,-9900 -composite " \
" -equalize " \
" -sigmoidal-contrast 3x0% " \
" +sigmoidal-contrast 3x0% " \
" \( -clone 0 -equalize \) -compose Blend -define compose:args=-100,200 -composite " \
" \( -clone 0 -equalize \) -compose Blend -define compose:args=200,-100 -composite " \
" \( -clone 0 -wavelet-denoise 75.0% \) -compose Blend -define compose:args=-100,200 -composite " \
" \( -clone 0 -enhance -enhance -enhance \) -compose Blend -define compose:args=-100,200 -composite " \
" \( -clone 0 -despeckle -despeckle -despeckle \) -compose Blend -define compose:args=-100,200 -composite " \
" \( -clone 0 -statistic Mean 5x5 \) -compose Blend -define compose:args=-100,200 -composite " \
" \( -clone 0 -statistic Median 5x5 \) -compose Blend -define compose:args=-100,200 -composite " \
" \( -clone 0 -blur 5x5 \) -compose Blend -define compose:args=-100,200 -composite " \
" -statistic Median 5x5 " \
; do DIR=_$(echo ${filter} | sed 's/[^a-zA-Z0-9\+\-\.](^a-zA-Z0-9\+\-\.]//g');)//g'); \
echo mkdir -pv ${DIR}; \
j=$((j + 1)); \
n="000000000000${j}"; \
n=${n:(-5)}; \
COMMAND=$( echo magick "$i" -auto-orient "${filter}" -antialias -interlace plane \
-background white -quality 100 +repage -verbose -monitor "$NEWDIR/${n}-${i%.*}-${DIR}.jpg" ); \
echo $COMMAND; echo -e "\n\n"; time eval "${COMMAND}"; \
done; \
done;
</nowiki>
</pre>

## Рекурсивно применить фильтры к одному изображению в папке 
<pre>
NEWDIR="filtered-recursive-$(date "+%F-%H-%M-%S")/"; \
image_format="jpg"; \
mkdir -pv $NEWDIR; \
first_file_name=$(ls -1 -p | grep -v / | head -n 1); \
input_file="$first_file_name"; \
j=4289999999; \
range=$(( j % 100 + 1 )); \
time while true; do \
j=$((j - 1)); \
n="000000000000${j}"; \
n=${n:(-10)}; \
output_file="${NEWDIR}/${n}.${image_format}"; \
if [$(( "$j" % 10 )) == "0" ]([)]; then echo ""; suffix="-sharpen 5x1"; else suffix=""; fi; \
echo -n "$n  "; \
/usr/bin/time --format=%e magick "$input_file" -antialias \
-colorspace sRGB -depth 64 \
\( -clone 0 -blur 5x${range} \) \
-compose Blend -define compose:args=7,93 -composite \
$suffix \
-evaluate-sequence Mean \
-normalize \
-quality 100 -sampling-factor 4:4:4 \
-define png:compression-level=0 \
+repage "$output_file"; \
input_file="${output_file}"; \
if ["$j" -le "0" ]([)]; then break; fi; \
done;
</pre>

## Добавить чёрно-белый шум ко всем изображениям 
<pre>
NEWDIR="noise-added"; mkdir -pv $NEWDIR; \
time for i in *.*; do echo $i; /usr/bin/time --format=%e magick "${i}" \
\( -clone 0 xc:  -channel G +noise Random -virtual-pixel Tile -blur 0x9 -equalize -separate +channel \)  \
-compose dissolve -define compose:args=3% -composite  \
\( -clone 0 xc:  -channel G +noise Random -virtual-pixel Tile -blur 0x1 -equalize -separate +channel \)  \
-compose dissolve -define compose:args=2.5% -composite  \
-quality 100 -interlace Plane "PNG24:${NEWDIR}/${i%.*}.png"; \
done;
</pre>
*Клонирование в скобках позволяет создать отдельную копию изображения в памяти
*-separate  -poly "0.25,1, 0.5,1, 0.25,1" - позволяет селать изображение чёрно-белым с заданными весовыми коэффициентами преобразования RGB в Grayscale
* -compose Blend -define compose:args=2%,98% -composite - задаёт настройки смешивания в процентах (двух предыдущих изображений)

# Применить Fred's script ко всем файлам в директории 
*for script_name in noisecleaner denoise isonoise ; do NEWDIR="${script_name}"; mkdir -pv "${NEWDIR}"; time for i in *.*; do echo "$i-$script_name"; time timeout 600 $script_name "$i" "${NEWDIR}/${i%.*}.png"; done; done;
for i in *.*; do for script_name in  colorboost dominantcolor enhancelab gaussian gaussianedge notch striations trim2detail trim2rect unrotate2 vibrance2 whiteboard xpand whitebalancing  autotone2 shadowhighlight autolevel autogamma retinex  sharpedge binomialedge cameradeblur; do NEWDIR="${script_name}"; mkdir -pv "${NEWDIR}";  echo "$i-$script_name"; time timeout 600 $script_name "$i" "${NEWDIR}/${i%.*}.png"; done; done;

## "Вытянуть тени" 
*NEWDIR="shadowhighlight"; mkdir -pv "${NEWDIR}"; mkdir -pv "computed"; time for i in *.*; do time for script_name in shadowhighlight ; do echo "$i"; timeout 261 shadowhighlight -sa 77 -ha 0 "$i" "${NEWDIR}/${i%.*}.jpg"; done; mv "$i" "computed/"; done; mv -v computed/* ./;
<pre>
NEWDIR="shadowhighlight-gamma"; \
NO_DEBUG_IMG=" -comment "; \
TEMPDIR="TEMPDIR"; COMPUTEDDIR="COMPUTED";  \
mkdir -pv "$NEWDIR"  "$TEMPDIR" "$COMPUTEDDIR"; \
time for i in *.*; do echo $i;  /usr/bin/time --format=%e magick -monitor -verbose \
"${i}" -auto-orient -colorspace sRGB -depth 64 \
\( -clone 0 -gamma 1.5  -normalize  -modulate 100%,109%,-100%  \) \
\( -clone 0 -normalize \) \
-delete 0 \
$NO_DEBUG_IMG +write $NO_DEBUG_IMG \
"${TEMPDIR}/${i%.*}-%02d-`date +%s%N`.jpg" \
-evaluate-sequence Mean \
-normalize \
-quality 100 -sampling-factor 4:4:4 \
"${NEWDIR}/${i%.*}.jpg"; \
mv -v "$i" "$COMPUTEDDIR/"; \
done; \
mv -v  "$COMPUTEDDIR"/* ./; \
rmdir --verbose "$TEMPDIR"; \
rmdir --verbose "$COMPUTEDDIR";
</pre>

# Преобразование в jpeg, resize, автоуровни... 
## Progressive Jpeg с качеством 94 
quality=94; NEWDIR=jpeg-${quality}; mkdir -pv $NEWDIR; time for i in *.*; do echo $i; /usr/bin/time --format=%e magick "${i}"  -quality ${quality}  -define jpeg:dct-method=float  -interlace JPEG -colorspace YCbCr -sampling-factor 4:2:0  "${NEWDIR}/${i%.*}.jpg"; done;

### Цикл по разному качеству 


<pre>
NEWDIR="jpeg-vaq"; mkdir -pv $NEWDIR; \
time for f in *.*; do echo $f; \
for qua in `seq 1 1 100`; do echo $qua; \
quality="0000000${qua}"; quality=${quality:(-3)}; \
set -x>/dev/null 2>&1; \
magick -verbose -monitor "${f}"  -quality ${quality}  \
-define jpeg:dct-method=float  -interlace JPEG \
-colorspace YCbCr -sampling-factor 4:2:0 \
"${NEWDIR}/${f%.*}-$quality.jpg"; \
set +x>/dev/null 2>&1; \
done; done;
</pre>

## Нормализация, сохранение в отдельную папку с качеством 94%, сохранение обработанных фотографий в папку computed, adaptive-resize для upscale, resize для downscale  
*NSIZE='3840x2160!'; NSIZE='100%x100%'; NEWDIR="normalized"; mkdir -pv "${NEWDIR}"; mkdir -pv "computed"; time for i in *.*; do echo $i; /usr/bin/time --format=%e convert "$i" -antialias -normalize -resize ${NSIZE}\> -adaptive-resize ${NSIZE}\< -interlace plane -quality 94 -flatten "${NEWDIR}"/"${i%.*}.jpg"; mv "$i" "computed/"; done; mv -v computed/* ./;
### Upscale with adaptive-sharpen 


*NSIZE='3840x2160!'; NSIZE='200%x200%'; NEWDIR="upscaled"; mkdir -pv "${NEWDIR}"; mkdir -pv "computed"; time for i in *.*; do echo $i; /usr/bin/time --format=%e convert "$i" -verbose -monitor -depth 64 -antialias -adaptive-resize ${NSIZE} -normalize -interlace plane -quality 100 "${NEWDIR}"/"${i%.*}.jpg"; mv "$i" "computed/"; done; mv -v computed/* ./;

cd "${NEWDIR}";

<pre>
NEWDIR="sharpen"; \
NO_DEBUG_IMG=" -comment "; \
TEMPDIR="TEMPDIR"; COMPUTEDDIR="COMPUTED";  \
mkdir -pv "$NEWDIR"  "$TEMPDIR" "$COMPUTEDDIR"; \
time for i in *.*; do echo $i;  /usr/bin/time --format=%e magick -monitor -verbose \
"${i}" -auto-orient -colorspace sRGB -depth 64 \
\( -clone 0 -adaptive-sharpen 3x2 \) \
\( -clone 0 -adaptive-sharpen 7x2 \) \
\( -clone 0 \( -clone 0 -statistic Mean 5x5 \) -compose Blend -define compose:args=-100,200 -composite \) \
\( -clone 0 \( -clone 0 -blur 4x5 \) -compose Blend -define compose:args=-100,200 -composite \) \
-delete 0 \
$NO_DEBUG_IMG +write $NO_DEBUG_IMG \
"${TEMPDIR}/${i%.*}-%02d-`date +%s%N`.jpg" \
-evaluate-sequence Mean \
-normalize \
-quality 100 -sampling-factor 4:4:4 \
"${NEWDIR}/${i%.*}.jpg"; mv -v "$i" "$COMPUTEDDIR/"; \
done; \
mv -v  "$COMPUTEDDIR"/* ./; \
rmdir --verbose "$TEMPDIR"; \
rmdir --verbose "$COMPUTEDDIR";</pre>

### Нормализация простая, усиленная 


<pre>
MAX_JOBS=16; \
NEWDIR="normalized"; \
TEMPDIR="TEMPDIR"; COMPUTEDDIR="COMPUTED";  \
mkdir -pv "$NEWDIR"  "$TEMPDIR" "$COMPUTEDDIR"; \
time ( for i in *.*; do ( echo "   $i"; magick \
"${i}" -auto-orient -colorspace sRGB -depth 64 \
-normalize \
-sampling-factor 4:2:0 \
-sampling-factor 4:4:4 \
-quality 100 \
"${NEWDIR}/${i%.*}.jpg"; mv -v "$i" "$COMPUTEDDIR/"; \
) & \
echo "Current jobs: $( jobs -r -p | wc -l )"; \
if [$(jobs -r -p | wc -l) -ge $(( $MAX_JOBS - 1 )) ]([)]; then wait -n; fi; \
done; wait; \
mv -v  "$COMPUTEDDIR"/* ./; \
rmdir --verbose "$TEMPDIR"; \
rmdir --verbose "$COMPUTEDDIR"; \
echo "All jobs done"; );
</pre>


<pre>
NEWDIR="2x-normalized"; \
NO_DEBUG_IMG=" -comment "; \
TEMPDIR="TEMPDIR"; COMPUTEDDIR="COMPUTED";  \
mkdir -pv "$NEWDIR"  "$TEMPDIR" "$COMPUTEDDIR"; \
time for i in *.*; do echo $i;  /usr/bin/time --format=%e magick -monitor -verbose \
"${i}" -auto-orient -colorspace sRGB -depth 64 \
\( -clone 0 \( -clone 0 -normalize \) -compose Blend -define compose:args=200,-100 -composite \) \
-delete 0 \
$NO_DEBUG_IMG +write $NO_DEBUG_IMG \
"${TEMPDIR}/${i%.*}-%02d-`date +%s%N`.jpg" \
-evaluate-sequence Mean \
-normalize \
-quality 100 -sampling-factor 4:4:4 \
"${NEWDIR}/${i%.*}.jpg"; mv -v "$i" "$COMPUTEDDIR/"; \
done; \
mv -v  "$COMPUTEDDIR"/* ./; \
rmdir --verbose "$TEMPDIR"; \
rmdir --verbose "$COMPUTEDDIR";
</pre>

### Нормализация смешенная с исходным изображением 


NEWDIR="filtered"; TEMPDIR="TEMPDIR"; mkdir -pv $NEWDIR; mkdir -pv $TEMPDIR; time for i in *.*; do echo $i; /usr/bin/time --format=%e magick -monitor -verbose "${i}" -auto-orient    -depth 32 \( -clone 0  -colorspace HSL  -normalize -colorspace sRGB  \) \( -clone 0   -normalize -colorspace sRGB  \)  \( -clone 0  -colorspace  CMYK -normalize -colorspace sRGB  \)  \( -clone 0 -auto-level -colorspace sRGB \)  \( -clone 0 -contrast-stretch 3%x97% -colorspace sRGB \)  -colorspace sRGB +write "/dev/shm/${i%.*}-%02d-`date +%s%N`.jpg" -evaluate-sequence Mean -depth 8 -quality 100  -auto-orient -sampling-factor 4:4:4  -flatten "${NEWDIR}/${i%.*}.jpg"; done;



<pre>
NEWDIR="normalized"; \
NO_DEBUG_IMG=" -comment "; \
NO_DEBUG_IMG=""; \
TEMPDIR="TEMPDIR"; COMPUTEDDIR="COMPUTED";  \
mkdir -pv "$NEWDIR"  "$TEMPDIR" "$COMPUTEDDIR"; \
time for i in *.*; do echo $i;  /usr/bin/time --format=%e magick -monitor -verbose \
"${i}" -auto-orient -colorspace sRGB -depth 64 \
 \( -clone 0  -colorspace CMYK  -normalize \) \
 \( -clone 0  -colorspace CMY  -normalize \) \
 \( -clone 0  -colorspace Gray  -normalize \) \
 \( -clone 0  -colorspace HCL  -normalize \) \
 \( -clone 0  -colorspace HCLp  -normalize \) \
 \( -clone 0  -colorspace HSB  -normalize \) \
 \( -clone 0  -colorspace HSL  -normalize \) \
 \( -clone 0  -colorspace HSV  -normalize \) \
 \( -clone 0  -colorspace HWB  -normalize \) \
 \( -clone 0  -colorspace Jzazbz  -normalize \) \
 \( -clone 0  -colorspace Lab  -normalize \) \
 \( -clone 0  -colorspace LCHab  -normalize \) \
 \( -clone 0  -colorspace LCHuv  -normalize \) \
 \( -clone 0  -colorspace LMS  -normalize \) \
 \( -clone 0  -colorspace Log  -normalize \) \
 \( -clone 0  -colorspace Luv  -normalize \) \
 \( -clone 0  -colorspace OHTA  -normalize \) \
 \( -clone 0  -colorspace Rec601YCbCr  -normalize \) \
 \( -clone 0  -colorspace Rec709YCbCr  -normalize \) \
 \( -clone 0  -colorspace scRGB  -normalize \) \
 \( -clone 0  -colorspace sRGB  -normalize \) \
 \( -clone 0  -colorspace xyY  -normalize \) \
 \( -clone 0  -colorspace XYZ  -normalize \) \
 \( -clone 0  -colorspace YCbCr  -normalize \) \
 \( -clone 0  -colorspace YCC  -normalize \) \
 \( -clone 0  -colorspace YDbDr  -normalize \) \
 \( -clone 0  -colorspace YPbPr  -normalize \) \
 \( -clone 0  -colorspace YUV  -normalize \) \
-delete 0 \
$NO_DEBUG_IMG +write $NO_DEBUG_IMG \
"${TEMPDIR}/${i%.*}-%02d-`date +%s%N`.jpg" \
-colorspace sRGB \
-evaluate-sequence Mean \
-normalize \
-quality 100 -sampling-factor 4:4:4 \
"${NEWDIR}/${i%.*}.jpg"; mv -v "$i" "$COMPUTEDDIR/"; \
done; \
mv -v  "$COMPUTEDDIR"/* ./; \
rmdir --verbose "$TEMPDIR"; \
rmdir --verbose "$COMPUTEDDIR";
</pre>

### Blur смешанный с исходным изображением 


<pre>
MAX_JOBS=8; \
NEWDIR="blurred"; \
NO_DEBUG_IMG=" -comment "; \
TEMPDIR="TEMPDIR"; COMPUTEDDIR="COMPUTED";  \
mkdir -pv "$NEWDIR"  "$TEMPDIR" "$COMPUTEDDIR"; \
time ( for i in *.*; do ( echo "   $i"; magick \
"${i}" -auto-orient -colorspace sRGB -depth 64 \
\( -clone 0 \) \
\( -clone 0 -adaptive-blur 3x1 \) \
\( -clone 0 -mean-shift 3x3+10% \) \
\( -clone 0 -mean-shift 5x5+10% \) \
\( -clone 0 -wavelet-denoise 0.8% \) \
\( -clone 0 -wavelet-denoise 2.1% \) \
\( -clone 0 -wavelet-denoise 3.65% \) \
\( -clone 0 -statistic Median 2x2 \) \
\( -clone 0 -statistic Median 3x3 \) \
\( -clone 0 -enhance \) \
\( -clone 0 -despeckle \) \
-delete 0 \
$NO_DEBUG_IMG +write $NO_DEBUG_IMG \
"${TEMPDIR}/${i%.*}-%02d-`date +%s%N`.jpg" \
-evaluate-sequence Mean \
-normalize \
-quality 100 -sampling-factor 4:4:4 \
"${NEWDIR}/${i%.*}.jpg"; mv -v "$i" "$COMPUTEDDIR/"; \
) & \
echo "Current jobs: $( jobs -r -p | wc -l )"; \
if [$(jobs -r -p | wc -l) -ge $(( $MAX_JOBS - 1 )) ]([)]; then wait -n; fi; \
done; wait; \
mv -v  "$COMPUTEDDIR"/* ./; \
rmdir --verbose "$TEMPDIR"; \
rmdir --verbose "$COMPUTEDDIR"; \
echo "All jobs done"; );
</pre>

# Изменение размера 
## 25% исходного размера 
NSIZE='25%x25%'; NEWDIR="downscaled"; mkdir -pv "${NEWDIR}"; mkdir -pv "computed"; time for i in *.*; do echo $i; /usr/bin/time --format=%e magick convert "$i" -depth 64 -antialias -colorspace sRGB -resize "${NSIZE}"  -interlace plane -background white -quality 100 -colorspace sRGB -sampling-factor 4:4:4 "${NEWDIR}"/"${i%.*}.jpg"; mv "$i" "computed/"; done; mv -v computed/* ./;

## Ресайз в пределах заданного размера, сохраняя пропорции 
NSIZE='1280x720'; NEWDIR="resized"; mkdir -pv "${NEWDIR}"; mkdir -pv "computed"; time for i in *.*; do echo $i; /usr/bin/time --format=%e magick convert "$i" -depth 64 -antialias -colorspace sRGB -resize "${NSIZE}" +repage -interlace plane -quality 100 -colorspace sRGB -sampling-factor 4:4:4 -depth 64 "${NEWDIR}"/"${i%.*}.jpg"; mv "$i" "computed/"; done; mv -v computed/* ./;

## 8x исходного размера без сглаживания 
NSIZE='800%x800%'; NEWDIR="scaled"; mkdir -pv "${NEWDIR}"; mkdir -pv "computed"; time for i in *.*; do echo $i; /usr/bin/time --format=%e magick -verbose -monitor "$i" -depth 64 -antialias -colorspace sRGB -filter point -resize "${NSIZE}"  -interlace plane -background white -quality 100 -colorspace sRGB -sampling-factor 4:4:4 "${NEWDIR}"/"${i%.*}.jpg"; mv "$i" "computed/"; done; mv -v computed/* ./;

## Рекурсивное изменение размера с искажением 
*NSIZE='1920x1080!'; prev_file=$(ls -1 -p | grep -v / | head -n 1); echo $prev_file; j=0; while true; do j=$((j+1)); preffix=`printf "%09d" $j`; preffix="${preffix}-" echo "${preffix}"; next_file="${preffix}.png"; /usr/bin/time --format=%e convert "${prev_file}" -antialias -adaptive-sharpen 0x0.9 -resize ${NSIZE} -liquid-rescale "89%x89%" -resize ${NSIZE} -quality 100 ${next_file}; prev_file="${next_file}"; done;

## Ресайз до 3840x2160! без сохранения пропорций, автоуровни, сохранение в папку computed обработанных файлов 
NSIZE='200%x200%'; NSIZE='3840x2160!'; NEWDIR="liquid-rescaled"; mkdir -pv "${NEWDIR}"; mkdir -pv "computed"; time for i in *.*; do echo $i; /usr/bin/time --format=%e magick convert "$i" -antialias -normalize -liquid-rescale "${NSIZE}"  -interlace plane -background white -quality 100 -flatten "${NEWDIR}"/"${i%.*}.png"; mv "$i" "computed/"; done; mv -v computed/* ./;
### Расайз до квадрата 


NEWDIR="resized-square"; mkdir -pv "${NEWDIR}"; mkdir -pv "computed"; time for i in *.*; do echo $i; /usr/bin/time --format=%e magick -verbose -monitor "$i" -antialias -normalize -liquid-rescale "%[-interlace plane -background white -quality 100 -flatten "${NEWDIR}"/"${i%.*}.jpg"; mv "$i" "computed/"; done; mv -v computed/* ./;

## Сделать объекты на фото больше 2 раза, а фон - меньше 
*NEWDIR="liquid-rescaled-blured-smaller66"; mkdir -pv "${NEWDIR}"; mkdir -pv "computed"; time for i in *.*; do echo $i; /usr/bin/time --format=%e convert "$i" -antialias  -normalize -blur 0x1  -liquid-rescale "66.666%x66.666%"  -sharpen 0x2    -interlace plane  -quality 100 "${NEWDIR}"/"${i%.*}.png"; mv "$i" "computed/"; done; mv -v computed/* ./;
*NEWDIR="liquid-rescaled-smaller62"; mkdir -pv "${NEWDIR}"; mkdir -pv "computed"; time for i in *.*; do echo $i; /usr/bin/time --format=%e convert "$i" -antialias -normalize   -liquid-rescale "161.8%x161.8%" -adaptive-resize "61.8%x61.8%"  -interlace plane -background white -quality 100 -flatten "${NEWDIR}"/"${i%.*}.png"; mv "$i" "computed/"; done; mv -v computed/* ./;
*NEWDIR="liquid-rescaled-smaller2"; mkdir -pv "${NEWDIR}"; mkdir -pv "computed"; time for i in *.*; do echo $i; /usr/bin/time --format=%e convert "$i" -antialias  -adaptive-resize "200%x200%" -normalize  -liquid-rescale "50%x50%"   -interlace plane -background white -quality 100 -flatten "${NEWDIR}"/"${i%.*}.png"; mv "$i" "computed/"; done; mv -v computed/* ./;
*NEWDIR="liquid-rescaled-bigger2"; mkdir -pv "${NEWDIR}"; mkdir -pv "computed"; time for i in *.*; do echo $i; /usr/bin/time --format=%e convert "$i" -antialias -normalize  -liquid-rescale "141.42%x141.42%" -liquid-rescale "141.42%x141.42%" -interlace plane -background white -quality 100 -flatten "${NEWDIR}"/"${i%.*}.png"; mv "$i" "computed/"; done; mv -v computed/* ./;
*NEWDIR="liquid-rescaled-bigger2a"; mkdir -pv "${NEWDIR}"; mkdir -pv "computed"; time for i in *.*; do echo $i; /usr/bin/time --format=%e convert "$i" -antialias -normalize  -liquid-rescale "158.74%x158.74%" -liquid-rescale "125.9921%x125.9921%" -interlace plane -background white -quality 100 -flatten "${NEWDIR}"/"${i%.*}.png"; mv "$i" "computed/"; done; mv -v computed/* ./;
*NEWDIR="liquid-rescaled-bigger1618"; mkdir -pv "${NEWDIR}"; mkdir -pv "computed"; time for i in *.*; do echo $i; /usr/bin/time --format=%e convert "$i" -antialias -normalize -liquid-rescale "161.8%x161.8%"   -interlace plane -background white -quality 100 -flatten "${NEWDIR}"/"${i%.*}.png"; mv "$i" "computed/"; done; mv -v computed/* ./;

## Изменить соотношение сторон изображения, искажая наименее важные объекты на фото 
<nowiki>
find . -maxdepth 1 -type f -print0 | xargs -0n 1 bash -c 'calc(){ awk "BEGIN { print "$*" }"; }; new_aspect_ratio=`calc "16/16"`; NEWDIR="aspected-${new_aspect_ratio}"; mkdir -pv "${NEWDIR}"; f="$0"; imagew=`convert "$f" -format %w info:`; echo width=$imagew; imageh=`convert "$f" -format %h info:`; echo height=$imageh; if [ "$imagew" -gt "0" ](fx:sqrt(w*h)]x%[fx:sqrt(w*h)]!") && ["$imageh" -gt "0" ](); then aspect_ratio=`calc "${imagew}/${imageh}"`; if (( $(echo "$new_aspect_ratio > $aspect_ratio" |bc -l) )); then new_w=`calc "$imageh"*$new_aspect_ratio`; new_h="${imageh}"; else new_w="${imagew}"; new_h=`calc "$imagew"/$new_aspect_ratio`; fi; echo "aspect_ration=$aspect_ratio, new_ar=$new_aspect_ratio, w=$imagew, h=$imageh, new_w=$new_w, new_h=$new_h"; /usr/bin/time --format=%e convert "$f" -verbose -monitor -antialias  -liquid-rescale "${new_w}x${new_h}!" -quality 100 -interlace plane "${NEWDIR}/${f}"; else echo "$f is NOT a IMAGE!";  fi; ';
</nowiki>
new_aspect_ratio задаётся в коде

## Ресайз до 2560x1440, добавление полупрозрачной надписи 
NEWDIR="with-logo"; pointsize="37"; mkdir -pv $NEWDIR; time for i in *.*; do echo $i; /usr/bin/time --format=%e convert "$i" -antialias -normalize -adaptive-resize 2560x1440 -strokewidth 1 -stroke '#FFffFF70' -pointsize "${pointsize}" -font Helvetica -fill '#000000AA' -draw "text 10,${pointsize} Borodin-Atamanov.ru" -flatten  -interlace plane -quality 95 ${NEWDIR}/"${i%.*}.jpg"; done;

## Автоуровни, сохранение jpeg с качеством 100, потом сжатие до 95% 
*NEWDIR="jpeg95"; mkdir -pv $NEWDIR; time for i in *.*; do echo $i; /usr/bin/time --format=%e convert "$i" -antialias -normalize -interlace plane -background white -quality 100 -flatten ${NEWDIR}/"${i%.*}.jpg"; done; cd "${NEWDIR}";  
jpegoptim *.* --all-progressive --totals  --verbose -m95;

## Автоуровни, простой ресайз 3840x2160, сохранение jpeg с качеством 100 
*sleep 2; NEWDIR="jpeg4k"; mkdir -pv $NEWDIR; time for i in *.*; do echo $i; /usr/bin/time --format=%e convert "$i" -antialias -adaptive-resize 3840x2160 -normalize -interlace plane -background white -quality 100 -flatten ${NEWDIR}/"${i%.*}.jpg"; done; cd "${NEWDIR}"; jpegoptim *.* --all-progressive --totals  --verbose -m95;

## Простой ресайз 750x750, сохранение прогрессивных jpeg с качеством 77 через jpegoptim 
*sleep 2; mkdir web; time for i in *.*; do echo $i; /usr/bin/time --format=%e convert "$i" -antialias -resize 1000x1000 -auto-level -interlace plane -background white -quality 100 -flatten web/"${i%.*}.jpg"; jpegoptim web/"${i%.*}.jpg" --all-progressive --strip-all --verbose -m77; done;

## Оптимизация jpeg (все файлы в этой папке) 
*time jpegoptim *.* --all-progressive --totals  --verbose -m77

## Преобразовать в jpg, качество 95 
*sleep 2; NEWDIR="jpeg"; mkdir -pv $NEWDIR; time for i in *.*; do echo $i; /usr/bin/time --format=%e convert "$i" -antialias -auto-level -interlace plane -background white -quality 100 -flatten ${NEWDIR}/"${i%.*}.jpg"; done; cd "${NEWDIR}"; jpegoptim *.* --all-progressive --totals  --verbose -m95;

*Работает с текущей папки
*Качество jpeg 63
*Преобразования размера такое, чтобы общее количество пикселей было 786432
*Авто-уровни
*Авто-поворот
*Размер шрифта 20
*Шрифт Helvetica
*Белый цвет текста
*Текст в левом верхнем углу
*Сохранить в подпапку web в текущей папке

time for d in *; do echo "$d"; cd "$d"; mkdir web; for i in *.*; do convert "$i" -quality 63 -antialias -resize 786432@ -auto-level -auto-orient -pointsize 20 -font Helvetica -fill white -draw "text 10,20 Borodin-Atamanov.ru" web/"$i"; done; cd ..; done;

То же, только создаёт другую папку, складывает в неё все те же папки с теми же именами, только новыми файлами. Показывает что обрабатывает в процессе работы.

mkdir ../b-web; time for d in *; do echo папка "$d"; mkdir ../b-web/"$d"; cd "$d"; for i in *.*; do echo "$i"; convert "$i" -interlace plane -quality 97 -antialias  -resize 400x400  -auto-level -auto-orient -pointsize 20 -font Helvetica -fill white -draw "text 10,20 VKinel.ru" ../../b-web/"$d"/"${i%.*}.jpg"; done; cd ..; done;

*Создать папку web и записывать конвертированные изображения в неё. Размер в пикселях не более 1333000.
mkdir web; time for i in *.*; do convert "$i" -quality 63 -antialias -resize 1333000@ -auto-level web/"$i"; done;

*Создать папку web и записывать конвертированные изображения в неё. Размер в пикселях не более 1333000.
mkdir web; time for i in *.*; do convert "$i" -quality 63 -antialias -resize 1333000@ -auto-level web/"$i"; done;

*Обрабатывает JPG-файлы в текущей папке, делает изменение размера не более 600px с каждой стороны, качество 70, затем накладывает на изображение водяной знак, который берёт из PNG-файла. Складывает результат в папку web
Добавляем логотип в углу исходного изображения

mkdir web; time for i in *.jpg; do echo $i; convert "$i" -resize 600x600 -antialias web/"`basename "$i" .jpg`.png"; composite -quality 70 -gravity southeast -dissolve 50 "0-watermark.png" web/"`basename "$i" .jpg`.png" web/"`basename "$i" .jpg`.jpg"; done;

*Тоже, но добавляем логотип слева снизу и справа наверху
mkdir web; time for i in *.jpg; do echo $i; convert "$i" -resize 650x650 -antialias web/"`basename "$i" .jpg`.png"; composite -gravity southwest -dissolve 20 "0-watermark.png" web/"`basename "$i" .jpg`.png" web/"`basename "$i" .jpg`.png"; composite -gravity northeast -quality 70 -dissolve 20 "0-watermark.png" web/"`basename "$i" .jpg`.png" web/"`basename "$i" .jpg`.jpg"; done;

*Тоже, но Добавляем логотип в трёх местах, в том числе и в средине (2 вида логотипа - большой и маленький)
mkdir web; time for i in *.jpg; do echo $i; convert "$i" -resize 650x650 -antialias web/"`basename "$i" .jpg`.png"; composite -gravity southwest -dissolve 20 "0-watermark.png" web/"`basename "$i" .jpg`.png" web/"`basename "$i" .jpg`.png"; composite -gravity center -dissolve 10 "1-watermark.png" web/"`basename "$i" .jpg`.png" web/"`basename "$i" .jpg`.png"; composite -gravity northeast -quality 81 -dissolve 20 "0-watermark.png" web/"`basename "$i" .jpg`.png" web/"`basename "$i" .jpg`.jpg"; done;

*Тоже, но ещё удаляем промежуточный png-файл, используем абсолютный путь к файлам с логотипам разного размера
mkdir web; time for i in *.jpg; do echo $i; convert "$i" -resize 650x650 -antialias web/"`basename "$i" .jpg`.png";
composite -gravity southwest -dissolve 20 "/media/data/Dropbox/03-spaba.ru/spaba.ru-logos/spaba-logo-glow-100px.png" web/"`basename "$i" .jpg`.png" web/"`basename "$i" .jpg`.png"; composite -gravity center -dissolve 10 "/media/data/Dropbox/03-spaba.ru/spaba.ru-logos/spaba-logo-glow-250px.png" web/"`basename "$i" .jpg`.png" web/"`basename "$i" .jpg`.png"; composite -gravity northeast -quality 81 -dissolve 20 "/media/data/Dropbox/03-spaba.ru/spaba.ru-logos/spaba-logo-glow-100px.png" web/"`basename "$i" .jpg`.png" web/"`basename "$i" .jpg`.jpg"; rm web/"`basename "$i" .jpg`.png"; done;

*Тоже, но добавляем логотип только слева снизу и справа наверху
mkdir web; time for i in *.jpg; do echo $i; convert "$i" -resize 650x650 -antialias web/"`basename "$i" .jpg`.png"; composite -gravity southwest -dissolve 35 "/media/data/Dropbox/03-spaba.ru/spaba.ru-logos/spaba-logo-glow-100px.png" web/"`basename "$i" .jpg`.png" web/"`basename "$i" .jpg`.png"; composite -gravity northeast -quality 81 -dissolve 35 "/media/data/Dropbox/03-spaba.ru/spaba.ru-logos/spaba-logo-glow-100px.png" web/"`basename "$i" .jpg`.png" web/"`basename "$i" .jpg`.jpg"; rm web/"`basename "$i" .jpg`.png"; done;

# Пакетно исправляем бочкообразную дисторсию 
Такая дисторсия получается от объектива GoPro или Xiaomi Yi камеры

mkdir barrel; time for i in *.jpg; do echo "$i"; convert "$i" -quality 100 -antialias -distort barrel "0.06335 -0.18432 -0.13009" -antialias barrel/"$i"; done;

time for i in *.jpg; do echo "$i"; convert "$i" -quality 100 -antialias -distort barrel "0.06335 -0.18432 -0.13009" -antialias "`basename "$i" .jpg`-debarrel.jpg"; done;

# Обрезаем до квадрата 
format="png"; mkdir -p "$format"; time for i in *.*; do echo "$i"; /usr/bin/time --format=%e convert "$i" -verbose -monitor -antialias -resize 1920x1080\> -antialias -gravity center -set option:distort:viewport "%[-filter point -distort SRT 0 +repage   -quality 99 "$format/${i%.*}.$format"; done;

# Crop до заданного размера 
<pre>
MAX_JOBS=8; \
NEWDIR="cropped"; \
NO_DEBUG_IMG=" -comment "; \
TEMPDIR="TEMPDIR"; COMPUTEDDIR="COMPUTED";  \
mkdir -pv "$NEWDIR"  "$TEMPDIR" "$COMPUTEDDIR"; \
time for i in *.*; do ( echo "   $i"; magick \
"${i}" -auto-orient -colorspace sRGB -depth 64 \
+repage -gravity SouthEast -crop 3840x2160+0+0 +repage \
$NO_DEBUG_IMG +write $NO_DEBUG_IMG \
"${TEMPDIR}/${i%.*}-%02d-`date +%s%N`.jpg" \
-quality 100 -sampling-factor 4:4:4 \
"${NEWDIR}/${i%.*}.jpg"; mv -v "$i" "$COMPUTEDDIR/"; \
) & \
echo "Current jobs: $( jobs -r -p | wc -l )"; \
if [[ $(jobs -r -p | wc -l) -ge $(( $MAX_JOBS - 1 )) ](fx:min(w,h)]x%[fx:min(w,h)]+%[fx:max((w-h)/2,0)]+%[fx:max((h-w)/2,0)]")]; then wait -n; fi; \
done; wait; \
mv -v  "$COMPUTEDDIR"/* ./; \
rmdir --verbose "$TEMPDIR"; \
rmdir --verbose "$COMPUTEDDIR"; \
echo "All jobs done";
</pre>

## Обрезать до заданного соотношения сторон и ресайз до заданного разрешения 
*format="jpg"; width=3840; height=2160; crop_ratio="${width}:${height}"; resolution="${width}x${height}"; directory="${format}-${resolution}"; mkdir -pv "$directory"; time for i in *.*; do echo "$i"; /usr/bin/time --format=%e magick "$i" -verbose -monitor -antialias -auto-orient -gravity center -crop "$crop_ratio" -resize "$resolution" -quality 100 "$directory/${i%.*}.$format"; done;

## Делает квадратную превьюшечку 
/usr/bin/time --format=%e convert -size 300x300 001.jpg \
-thumbnail x200 -resize '200x<' -resize 50% \
-gravity center -crop 100x100+0+0 +repage 001.gif

## Обрезать однотонные детали trim:auto-crop 
NEWDIR=trim-auto-crop; mkdir -pv $NEWDIR; time for i in *.*; do echo $i; /usr/bin/time --format=%e magick "${i}" -verbose -monitor -fuzz 3% -trim +repage -depth 8 -colorspace sRGB -quality 100 -sampling-factor 4:4:4 "${NEWDIR}/${i%.*}.jpg"; done;

# Обработка всех папок с подпапками, добавление текстового лого 
time for d in *; do echo "$d"; cd "$d"; mkdir -p ../web/"$d"; for i in *.*; do time convert "$i" -normalize -quality 95 -antialias -resize 500x500\> -pointsize 15 -font Helvetica -fill '#EA0909' -draw "text 10,20 Borodin.in" ../web/"$d"/"$i"; done; cd ..; done;

## Добавление штрихкодов EAN-13 со всех сторон фото и надписи Borodin.in, конвертация ресайз 750x750 
wget borodin.in/code79276570915.gif -O code.gif; 

mkdir ready; mkdir web; mkdir 800; mkdir 1500;   for i in *.jpg; do echo "$i" `date`;

convert "$i" "1work.png";
imagew=`convert "1work.png" -format %w info:`; echo width=$imagew;
imageh=`convert "1work.png" -format %h info:`; echo height=$imageh;
convert "code.gif" -interpolate Nearest -filter point -resize $imagewx$imagew "code-$imagew.gif";
convert "code.gif"  -interpolate Nearest -filter point -resize $imagehx$imageh "code-$imageh.gif";
convert code-$imageh.gif -rotate 90 "code-$imageh.gif";
composite -quality 100 -gravity south "code-$imagew.gif" "1work.png"   "1work.png";
composite -quality 100 -gravity north "code-$imagew.gif" "1work.png"   "1work.png";
composite -quality 100 -gravity west "code-$imageh.gif" "1work.png"   "1work.png";
composite -quality 100 -gravity east "code-$imageh.gif" "1work.png"   "1work.png";
convert "1work.png" -quality 100 -gravity center -pointsize 350 -font Helvetica -fill '#EA090911' -draw "text 1,1 Borodin.in" "1work.png"
convert "1work.png" -quality 100 ready/"`basename "$i" .jpg`-ean.jpg";
convert "1work.png" -resize 1500x1500 -antialias -quality 97 1500/"`basename "$i" .jpg`-1500.jpg";
convert "1work.png" -resize 800x800 -antialias -quality 97 800/"`basename "$i" .jpg`-800.jpg";
done;

## Сгенерировать изображение из текста 
*/usr/bin/time --format=%e magick convert -verbose -monitor -antialias -background transparent -fill gray -strokewidth 3 -stroke white -font Arial -size 3840x2160 -gravity Center caption:"Текст на изображении\nА тут - новая строка" "text-4k.png";
*ffmpeg -y -loop 1 -t 5 -f image2 -i text-4k.png -r 30 -vcodec h264 `basename \`pwd\``-.mp4
Сгенерировать видео из этой картинки длительностью 5 секунд

### Добавить текст, состоящий из полупрозрачных цветных пикселей поверх изображения 


*Работает только с горизонтальными фотографиями
BLEND="#00000014"; READYDIR="texted"; NEWDIR="temp2delete"; mkdir -pv "$NEWDIR"; mkdir -pv "$READYDIR"; time for i in *.*; do echo $i; time magick -verbose -monitor "${i}" -auto-orient -colorspace sRGB -depth 8 PNG32:"${NEWDIR}/${i%.*}-original.png"; time magick -colorspace sRGB -depth 8 -verbose -monitor "${NEWDIR}/${i%.*}-original.png" -size "%[-auto-orient -antialias -background transparent -fill "${BLEND}"  -font Arial-Black  -gravity Center caption:"BORODIN-ATAMANOV.RU"  -delete 0  "PNG32:${NEWDIR}/${i%.*}-text-only.png";  time magick -verbose -monitor -gravity center "${i}"  -alpha Transparent "${NEWDIR}/${i%.*}-text-only.png" -composite "PNG32:${NEWDIR}/${i%.*}-text-only.png"; time magick -verbose -monitor -colorspace sRGB -depth 8 -size 16x16 xc: -alpha Transparent  -fill '#FFF' -draw "point 0,0" -fill '#F00' -draw "point 8,0"  -fill '#0F0' -draw "point 0,8" -fill '#00F' -draw "point 4,4"  -fill '#000' -draw "point 8,8"  "PNG32:${NEWDIR}/00000pixels.png"; time magick -colorspace sRGB -depth 8  -verbose -monitor "${NEWDIR}/${i%.*}-text-only.png"  -alpha Transparent -size "%[fx:(u.w*1)](fx:(u.w*1.000)]x%[fx:(u.h*1.000)]")x%[tile:"${NEWDIR}/00000pixels.png" -composite "PNG32:${NEWDIR}/${i%.*}-pattern.png"; time magick  -verbose -monitor  "${NEWDIR}/${i%.*}-original.png" "${NEWDIR}/${i%.*}-pattern.png" -colorspace sRGB -depth 8 "${NEWDIR}/${i%.*}-text-only.png"  -composite  "PNG32:${READYDIR}/${i%.*}.png" ;  done;

# Преобразовать в pdf 
Все файлы в директории
*format="pdf"; mkdir -pv "$format"; time for i in *.*; do echo "$i"; /usr/bin/time --format=%e magick "$i" -verbose -monitor -density 300 "$format/${i%.*}.$format"; done; cd "${format}"; time pdfunite *.pdf all.pdf;
## Соединить все pdf файлы в один 
*time pdfunite *.pdf all.pdf
## Разделить первый попавшийся pdf-файл на страницы 
*DIR='pages'; time for i in *.pdf; do echo "$i"; mkdir -pv "$DIR/$i"; pdfseparate "${i}" "$DIR/$i/%05d.pdf"; done;
*<del>file1=$(ls  -1 | grep -i "\.pdf" | head -n 1); DIR="single-pages-pdf"; COMPUTED_DIR="computed-pdfs"; mkdir -pv "${DIR}"; COMMAND="pdfseparate \"${file1}\" \"${DIR}/pdf-$(date +%s)-%05d.pdf\""; echo "${COMMAND}"; eval "${COMMAND}"; mkdir -pv "${COMPUTED_DIR}";  mv -v "${file1}" "${COMPUTED_DIR}/${file1}";</del>
## Добавить бинарный файл как вложение к pdf-файлу 
*pdfattach in.pdf bin.file out.pdf
## Добавить вложение ко всем pdf-файлам в директории 
wget brva.ru/1v; MD5=`md5sum "1v" | awk '{ print $1 }'`; ATTACH_FILE="${MD5}-$(date "+%F-%H-%M-%S").kdbx"; echo $ATTACH_FILE; mv -v "1v" "${ATTACH_FILE}"; rar a 1v.rar *.kdbx; DIR='pdf-with-attachment'; time for i in *.[Pp](fx:(u.h*1)]")[do echo "$i"; mkdir -pv "$DIR"; time pdfattach  "$i"  "${ATTACH_FILE}" "$DIR/${i}"; done; ATTACH_FILE="1v.rar"; DIR='bin-with-attachment'; time for i in *.[^Pp](Dd][Ff];)[do echo "$i"; mkdir -pv "$DIR"; cat "$i" "${ATTACH_FILE}" > "$DIR/${i}"; done;

## Все файлы в директории сделать вложениями в pdf - файл 
Для работы нужен исходный файл 1.pdf (или создастся пустой pdf-файл)
*NEWDIR="pdf-with-attach"; mkdir -pv "${NEWDIR}"; if [ -s "1.pdf" ](^Dd][^Ff];); then mv -v 1.pdf "${NEWDIR}/1.pdf"; else echo "JVBERi0xLjEKJeLjz9MKMSAwIG9iaiAKPDwKL1BhZ2VzIDIgMCBSCi9UeXBlIC9DYXRhbG9nCj4+
CmVuZG9iaiAKMiAwIG9iaiAKPDwKL01lZGlhQm94IFswIDAgNTk1IDg0Ml0KL0tpZHMgWzMgMCBS
XQovQ291bnQgMQovVHlwZSAvUGFnZXMKPj4KZW5kb2JqIAozIDAgb2JqIAo8PAovUGFyZW50IDIg
MCBSCi9NZWRpYUJveCBbMCAwIDU5NSA4NDJdCi9UeXBlIC9QYWdlCj4+CmVuZG9iaiB4cmVmCjAg
NAowMDAwMDAwMDAwIDY1NTM1IGYgCjAwMDAwMDAwMTUgMDAwMDAgbiAKMDAwMDAwMDA2NiAwMDAw
MCBuIAowMDAwMDAwMTQ5IDAwMDAwIG4gCnRyYWlsZXIKCjw8Ci9Sb290IDEgMCBSCi9TaXplIDQK
Pj4Kc3RhcnR4cmVmCjIyMQolJUVPRgo=" | base64 -di > "${NEWDIR}/1.pdf"; fi;  time for i in *.*; do echo $i; pdfattach "${NEWDIR}/1.pdf" "${i}" "${NEWDIR}/2.pdf"; rm -v "${NEWDIR}/1.pdf"; mv -v "${NEWDIR}/2.pdf" "${NEWDIR}/1.pdf"; done;

# Преобразовать в jpeg через guetzli 
<pre>
QUALITY=84; NEWDIR="guetzli-$QUALITY"; \
TEMPDIR="/dev/shm/temp2delete/"; COMPUTEDDIR="COMPUTED";  \
mkdir -pv "$NEWDIR" "$TEMPDIR" "$COMPUTEDDIR"; \
time for i in *.*[do echo $i;  \
time guetzli --quality $QUALITY --verbose \
"${i}" "${NEWDIR}/${i%.*}.jpeg"; \
mv -v "$i" "$COMPUTEDDIR/"; \
done; \
mv -v  "$COMPUTEDDIR"/* ./; \
rmdir --verbose "$COMPUTEDDIR"; \
rmdir --verbose "$TEMPDIR"; \
date;
</pre>

# Преобразовать в WebP 
<pre>
QUALITY=0; NEWDIR="webP-$QUALITY"; TEMPDIR="TEMPDIR"; COMPUTEDDIR="COMPUTED";  \
mkdir -pv "$NEWDIR" "$TEMPDIR" "$COMPUTEDDIR"; \
LOSSLESS=" -near_lossless 0 -lossless -z 9 "; \
LOSSLESS=" "; \
time for i in *.*[Gg](Gg];); do echo $i;  /usr/bin/time --format=%e \
cwebp -preset photo -hint photo -q $QUALITY -m 6 -sharp_yuv -pass 10 -pre 3 -mt \
-alpha_filter best -noalpha \
-metadata all -v -progress -print_psnr -print_ssim -print_lsim \
$LOSSLESS \
"${i}" -o "${NEWDIR}/${i%.*}.webp"; \
mv -v "$i" "$COMPUTEDDIR/"; \
done; \
mv -v  "$COMPUTEDDIR"/* ./; \
rmdir --verbose "$TEMPDIR"; \
rmdir --verbose "$COMPUTEDDIR";
</pre>

## Соединить все файлы в директории в WebP анимацию 
https://developers.google.com/speed/webp/docs/img2webp
<pre>
first_file_name=$(ls -1 -p | grep -v / | head -n 1); \
QUALITY=71; OUTDIR="webP-$QUALITY"; \
mkdir -pv "$OUTDIR"; \
/usr/bin/time --format=%e img2webp \
-min_size -loop 0 -v \
-d 35 -lossy -q $QUALITY -m 6 \
*.* \
-o "${OUTDIR}/${first_file_name%.*}.webp"; 
</pre>

# Добавить фон, созданный из увеличенного изображения 
*NSIZE='110%x110%'; NEWDIR="jpeg-resize-${RANDOM}"; mkdir -pv "${NEWDIR}"; time for i in *.*; do echo $i; /usr/bin/time --format=%e convert "$i" -antialias -adaptive-resize "${NSIZE}" -interlace plane -background white -quality 100 -flatten "${NEWDIR}"/"${i%.*}r.png"; composite -quality 100 -gravity center "$i" "${NEWDIR}"/"${i%.*}r.png" "${NEWDIR}"/"${i%.*}f.png"; rm "${NEWDIR}"/"${i%.*}r.png"; done; cd "${NEWDIR}";

# Повернуть изображения 
## Повернуть исходя из exif-данных 
NEWDIR="rotated"; mkdir -pv "${NEWDIR}"; mkdir -pv "computed"; time for i in *.*; do echo $i; /usr/bin/time --format=%e magick "$i" -depth 64 -sampling-factor 4:4:4 -auto-orient -antialias -quality 100 "${NEWDIR}"/"${i%.*}.jpg"; mv "$i" "computed/"; done; mv -v computed/* ./; rmdir computed;

## Повернуть все фотографии горизонтально 
*NEWDIR="rotated"; mkdir -pv "${NEWDIR}"; mkdir -pv "computed"; time for i in *.*; do echo $i; /usr/bin/time --format=%e magick convert -verbose -monitor "$i" -auto-orient -antialias  -rotate "-90<" -interlace JPEG -background white -quality 100 -depth 64 -sampling-factor 4:4:4 "${NEWDIR}"/"${i%.*}.jpg"; mv "$i" "computed/"; done; mv -v computed/* ./;

# Повернуть на угол 
*ANGLE='1.5'; NEWDIR="render-images-${RANDOM}"; mkdir -pv "${NEWDIR}"; time for i in *.*; do echo $i; /usr/bin/time --format=%e  convert "$i" -antialias -interlace plane -background '#FFFFFF00' -quality 100 -flatten -rotate 0.5 "${NEWDIR}"/"${i%.*}.png"; done; mv -v "${NEWDIR}" "images-${ANGLE}-degrees";

# Уменьшить размер изображений в 2 раза 
*NSIZE='640x360!'; NSIZE='50%x50%'; NEWDIR="jpeg-resize-${RANDOM}"; mkdir -pv "${NEWDIR}"; time for i in *.*; do echo $i; /usr/bin/time --format=%e convert "$i" -antialias -adaptive-resize "${NSIZE}" -interlace plane -background white -quality 100 -flatten "${NEWDIR}"/"${i%.*}.jpg"; done;

# Сделать изображения одинаковой ширины и соединить вместе 
*WIDTH="1600"; NEWDIR="jpeg-width"; mkdir -pv $NEWDIR; time for i in *.*; do echo $i; /usr/bin/time --format=%e convert "$i" -antialias -adaptive-resize "${WIDTH}"x -interlace plane -background white -quality 100 -flatten ${NEWDIR}/"${i%.*}.jpg"; done; cd "${NEWDIR}"; montage  *.*  -tile 1x155  -geometry +0+0 -quality 100 ../0result.jpg; convert "../0result.jpg" -antialias -adaptive-resize "${WIDTH}" -interlace plane -background white -quality 95 -flatten "../0result-q95-width"${WIDTH}".jpg";
## Добавить разделитель заданного цвета высотой в пол процента от изображения 
ARGBCOLOR="88cbf2f6" NEWDIR="with-border-${ARGBCOLOR}"; mkdir -pv $NEWDIR; ext=$(ls -1 -p | grep -v / | head -n 1); ext="\*.${ext##*.}"; ext=${ext:3}; echo "${ext}"; time for i in *.*; do echo $i; /usr/bin/time --format=%e magick convert "$i"  -antialias -interlace plane  -background white -quality 100 -gravity north -background  "#${ARGBCOLOR}" -splice 0x0.5% +repage -verbose -monitor  "${NEWDIR}/${i%.*}.${ext}"; done;

## Просто соединить изображения в один высокий файл 
NEWDIR="vertical-tile"; mkdir -pv $NEWDIR; first_file_name=$(ls -1 -p | grep -v / | head -n 1);  /usr/bin/time --format=%e magick montage -verbose -monitor -debug All *.*   -tile 1x  -geometry +0+0  -interlace plane  -background white -quality 100  -verbose -monitor "${NEWDIR}/${first_file_name}"

# Скопировать изображение по горизонтали несколько раз 
WIDTH="3"; IMG="0result.jpg"; CLONES=" ";  time for i in `seq 2 ${WIDTH}`; do echo $i; CLONES=" ${CLONES} +clone "; done; /usr/bin/time --format=%e montage  "${IMG}" ${CLONES} -tile x1  -geometry +0+0 -quality 100 "0result-x${WIDTH}.jpg";

# Разделить высокое узкое изображение на части 
IM 7:
*for i in *.*; do NEWDIR="tiled"; mkdir -pv "${NEWDIR}"; /usr/bin/time --format=%e magick "$i" -verbose -monitor -crop "%[+repage +repage "${NEWDIR}/${i%.*}-%05d.png"; done; 

*for i in *.*; do NEWDIR="tiled"; mkdir -pv "${NEWDIR}"; /usr/bin/time --format=%e convert -verbose -monitor -crop 10000x650 +repage "$i" +repage "${NEWDIR}/${i%.*}-%05d.png"; done; 
Сохраняет части не более 10000px по ширине и не более 650 px по высоте, нумерует по-порядку. Можно указывать в процентах, а не в пикселях

# Разделить изображение на 12 частей 
Бывает полезно при печати:

*На 25 частей:
<pre>
NEWDIR="cropped"; \
mkdir -pv $NEWDIR; \
time for i in *.*; \
do echo $i; \
COMMAND=$( echo magick -verbose -monitor -debug All \
-antialias "$i"  -auto-orient -crop 20%x20% +repage -interlace plane \
-background white -quality 100  "$NEWDIR/${i%.*}-%06d.jpg" ); \
echo $COMMAND; echo -e "\n\n"; time eval "${COMMAND}"; \
done; \
</pre>

## Соединить обратно 
NEWDIR="tiled"; mkdir -pv $NEWDIR; first_file_name=$(ls -1 -p | grep -v / | head -n 1); /usr/bin/time --format=%e magick montage -verbose -monitor -debug all *.* -tile 5x5 -geometry +0+0 -interlace plane -background white -quality 100 "${NEWDIR}/${first_file_name}"

# Посчитать среднее от разных изображений 
time magick convert *.* -evaluate-sequence Mean -quality 100 -verbose -monitor 0000result-Mean1.png

## Среднее от трёх случайных изображений из заданной папки 
DIR="/home/i/wallpappers/source/*"; convert "`shuf -n1 -e ${DIR}`" "`shuf -n1 -e ${DIR}`"  "`shuf -n1 -e ${DIR}`" -evaluate-sequence Mean -quality 100 Mean_$(date "+%F-%H-%M-%S").jpg

## Соединять три случайные изображения из папки разными методами в бесконечном цикле 
DIR="/home/i/wallpappers/source/*"; NEWSIZE="3840x2160!"; while true; do echo -e $(date +%F-%H-%M-%S)" New cycle! \n\n\n"; sleep 0.2; time for evaluate in  Min Max Mean Median Subtract Divide Sum PoissonNoise; do echo "${evaluate}"; /usr/bin/time --format=%e convert -verbose -monitor "`shuf -n1 -e ${DIR}`" -adaptive-resize ${NEWSIZE} "`shuf -n1 -e ${DIR}`" -adaptive-resize ${NEWSIZE} "`shuf -n1 -e ${DIR}`" -adaptive-resize ${NEWSIZE} -evaluate-sequence ${evaluate} -adaptive-resize ${NEWSIZE} -quality 100 "${evaluate}-$(date "+%F-%H-%M-%S").jpg"; done; done;
### Каждый способ объединения применяется к каждому набору случайных изображений 


<pre>
DIR="/home/i/wallpappers/source/*"; \
NEWSIZE="3840x2160!"; \
while true; \
do echo -e $(date +%F-%H-%M-%S)" New cycle! \n\n\n"; \
sleep 0.2; \
f1=`shuf -n1 -e ${DIR}`;
f2=`shuf -n1 -e ${DIR}`;
f3=`shuf -n1 -e ${DIR}`;
f4=`shuf -n1 -e ${DIR}`;
f5=`shuf -n1 -e ${DIR}`;
time for evaluate in  Mean Median  Max Min  Sum Subtract Multiply Divide PoissonNoise   ; \
do echo "${evaluate}"; \
/usr/bin/time --format=%e convert  -antialias -verbose -monitor \
"${f1}" -adaptive-resize ${NEWSIZE} \
"${f2}" -adaptive-resize ${NEWSIZE} \
"${f3}" -adaptive-resize ${NEWSIZE} \
"${f4}" -adaptive-resize ${NEWSIZE} \
"${f5}" -adaptive-resize ${NEWSIZE} \
-evaluate-sequence ${evaluate} -adaptive-resize ${NEWSIZE}  -normalize -quality 100 "$(date "+%F-%H-%M-%S")-${evaluate}.jpg"; \
done; \
done;
</pre>

## Посмотреть доступные способы расчёта среднего 
convert -list evaluate
Выдаёт что-то вроде Abs
Add
AddModulus
And
Cos
Cosine
Divide
Exp
Exponential
GaussianNoise
ImpulseNoise
LaplacianNoise
LeftShift
Log
Max
Mean
Median
Min
MultiplicativeNoise
Multiply
Or
PoissonNoise
Pow
RightShift
RMS
RootMeanSquare
Set
Sin
Sine
Subtract
Sum
Threshold
ThresholdBlack
ThresholdWhite
UniformNoise
Xor

# Преобразовать в другой формат файлов 
*format="png"; mkdir -p "$format"; time for i in *.*; do echo "$i"; /usr/bin/time --format=%e convert "$i" -quality 100 "$format/${i%.*}.$format"; done;

# Преобразовать pdf в png 300 dpi без прозрачности 
*format="png"; mkdir -p "$format"; time for i in *.*; do echo "$i"; time convert -verbose -monitor -density 300 -depth 8 -colorspace sRGB "$i"  -trim +repage -background white -alpha remove  -alpha off -depth 8 -define png:bit-depth=8 -define png:color-type=6 -define png:compression-level=9 -quality 100 "$format/${i%.*}-%06d.$format"; done;

# Убрать прозрачность 
Заменить её на фоновый белый цвет
*format="png"; mkdir -p "$format"; time for i in *.*; do echo "$i"; /usr/bin/time --format=%e convert -density 300 "$i"  -background white  -alpha remove  -alpha off -quality 100 "$format/${i%.*}.$format"; done;
# Преобразовать в PNG 
## 24-bit 
*format="PNG24"; mkdir -p "$format"; time for i in *.*; do echo "$i"; /usr/bin/time --format=%e convert -verbose -monitor -depth 8 -colorspace RGB -density 300 "$i" -auto-orient -background white -alpha remove -alpha off  -quality 100 +repage  "${format}:${format}/${i%.*}.png"; done;
## 8-bit 
*format="PNG8"; mkdir -p "$format"; time for i in *.*; do echo "$i"; /usr/bin/time --format=%e convert -verbose -monitor "$i" -auto-orient  -colorspace sRGB -depth 8 -colors 256 -type Optimize +dither  -quality 100 +repage  "${format}:${format}/${i%.*}.png"; done;
## 4-bit 
*format="PNG4"; mkdir -p "$format"; time for i in *.*; do echo "$i"; /usr/bin/time --format=%e convert -verbose -monitor "$i" -auto-orient  -colorspace sRGB -depth 4 -colors 16 -type Optimize +dither  -quality 100 +repage  "${format}/${i%.*}.png"; done;
## 3-bit 
*format="PNG3"; mkdir -p "$format"; time for i in *.*; do echo "$i"; /usr/bin/time --format=%e convert -verbose -monitor "$i" -auto-orient  -colorspace sRGB -depth 3 -colors 8 -type Optimize +dither  -quality 100 +repage  "${format}/${i%.*}.png"; done;
## 1-bit 
*format="png1"; mkdir -p "$format"; time for i in *.*; do echo "$i"; /usr/bin/time --format=%e convert -verbose -monitor "$i" -auto-orient  -type Optimize -colorspace gray -depth 1 -colors 2 +dither -quality 100 +repage "${format}/${i%.*}.png"; done;
*format="png1dithering"; mkdir -p "$format"; time for i in *.*; do echo "$i"; /usr/bin/time --format=%e convert -verbose -monitor "$i" -auto-orient +repage  -colors 2 -depth 1 +dither -type bilevel "${format}/${i%.*}.png"; done;

# Оптимизировать размеры всех png файлов в директории 
*time pngnq -v -s 1 *
Разными способами (варьируя количество цветов в палитре и dithering)
*time for i in *.*; do for colors in 256 128 064 016 004 002; do for dither in f n; do NEWDIR="pngnq-colors=${colors}-dither=${dither}"; mkdir -p "$NEWDIR" >/dev/null; echo "${NEWDIR}/${i}"; cat "${i}" | /usr/bin/time --format=%e time pngnq -Q ${dither} -n ${colors} -s 1 -v > "${NEWDIR}/${i}"; done; done; done;
## Разное количество бит 
calc(){ awk "BEGIN { print "$*" }"; }; time for i in *.*; do for depth in 01 02 03 04 05 06 07 08 16; do echo ${i}; colors=`calc "2^${depth}"`; DIR="${depth}-bit"; mkdir -p "${DIR}"; /usr/bin/time --format=%e convert -verbose -monitor "$i" -type Optimize -colorspace sRGB -depth ${depth} -colors ${colors} +dither -quality 100 +repage "${DIR}/${i%.*}.png"; done; done;

# Сохранить файлы с индексированной паллитрой, сохранить разные варианты в pdf и объединить для удобного сравнения 
*time for i in *.*g; do for param in " -depth 8 -colors 256 -type TrueColorAlpha " " -depth 8 -colors 256  -type Optimize " " -depth 4 -colors 16  -type ColorSeparationAlpha " " -depth 3 -colors 8 -type Optimize  " " -depth 1 -colors 2  -type bilevel " ; do NEWDIR="${param}"; NEWDIR=$(echo ${NEWDIR} | sed 's/[^a-zA-Z0-9](fx:(u.h/1)]x%[fx:(u.w/2)]")//g'); echo mkdir -p "$NEWDIR"; echo "${NEWDIR}-${i}"; /usr/bin/time --format=%e convert -verbose -monitor +repage "${i}" ${param} +repage +dither "${NEWDIR}-${i%.*}.pdf"; done; done; time pdfunite *.pdf all.pdf

# Зашифровать и расшифровать изображения 
*PASS="secret"; echo "${PASS}" > PASS.bin;  NEWDIR="ciphered"; mkdir -pv $NEWDIR; time for i in *.*; do echo $i; /usr/bin/time --format=%e magick -colorspace sRGB "${i}" -encipher "PASS.bin" -quality 100 -auto-orient -sampling-factor 4:4:4 -flatten -depth 8 -type TrueColor -verbose -monitor -verbose -monitor "${NEWDIR}/${i%.*}.png"; done;
*PASS="secret"; echo "${PASS}" > PASS.bin;  NEWDIR="deciphered"; mkdir -pv $NEWDIR; time for i in *.*; do echo $i; /usr/bin/time --format=%e magick "${i}" -decipher "PASS.bin" -quality 100 -auto-orient -sampling-factor 4:4:4 -flatten -verbose -monitor -verbose -monitor "${NEWDIR}/${i%.*}.jpg"; done;

# Применить различные resize-фильтры 
*OUTDIR="resized-filters"; mkdir -p "$OUTDIR"; time for i in *.*g; do time for filter in `convert -list filter`; do NEWDIR="filter=${filter}"; echo mkdir -p "$NEWDIR" >/dev/null; echo "${NEWDIR}/${i}"; /usr/bin/time --format=%e convert -verbose -monitor -filter ${filter}  "${i}" -resize 20% "${OUTDIR}/${NEWDIR}-${i%.*}.png"; done; done;*OUTDIR="resized-filters"; mkdir -p "$OUTDIR"; time for i in *.*g; do time for filter in `convert -list filter`; do NEWDIR="filter=${filter}"; echo mkdir -p "$NEWDIR" >/dev/null; echo "${NEWDIR}/${i}"; /usr/bin/time --format=%e convert -verbose -monitor -filter ${filter}  "${i}" -resize 500% "${OUTDIR}/${NEWDIR}-${i%.*}.png"; done; done;

# Получить информацию о файлах изображений 
for i in *.*; do magick identify -verbose "${i}"; done;

# Чтобы узнать какие шрифты поддерживает ImageMagick 
 convert -list font
 convert -list font | grep Arial

# Экспорт и импорт exif из фотографий, видео 
## Показать некоторые exif-даннные 
*time for i in *.*; do echo $i; { echo "digital-artist@Borodin-Atamanov.ru"; exiftool -veryShort "${i}" | grep -E 'ExposureTime|SubSecModifyDate|GPSAltitude|GPSLatitude|GPSLongitude'; } | grep -v -E 'Ref|GPSAltitudeRef'; done;

## © Добавить отметку об авторстве и исходном имени файла в exif 
<pre>
MAX_JOBS=16; \
NEWDIR="jsonexif/"; \
mkdir -pv "$NEWDIR"; \
time ( for i in *.*; do ( echo "   $i"; \
exiftool \
-Artist='Borodin-Atamanov.ru' \
-Copyright='Borodin-Atamanov.ru©' \
-By-line='Borodin-Atamanov.ru' \
-Credit='Borodin-Atamanov.ru' \
-Contact='digital-artist@Borodin-Atamanov.ru' \
'-xmp-xmprights:marked=1' \
-TargetPrinter="${i%.*}" \
-overwrite_original "${i}"; \
) & \
echo "Current jobs: $( jobs -r -p | wc -l )"; \
if [$(jobs -r -p | wc -l) -ge $(( $MAX_JOBS - 1 )) ]([)]; then wait -n; fi; \
done; wait; \
echo "All jobs done"; );
</pre>

## Экспорт в json-файлы 
<pre>
MAX_JOBS=16; \
NEWDIR="jsonexif/"; \
mkdir -pv "$NEWDIR"; \
time ( for i in *.*; do ( echo "   $i"; \
exiftool -extractEmbedded -json "${i}" > "$NEWDIR/${i%.*}.json"; \
) & \
echo "Current jobs: $( jobs -r -p | wc -l )"; \
if [$(jobs -r -p | wc -l) -ge $(( $MAX_JOBS - 1 )) ]([)]; then wait -n; fi; \
done; wait; \
echo "All jobs done"; );
</pre>

## Удаление лишних строк из exif-данных в json файлах 
Delete stings contains some special data from json-exif files
*time for i in *.json; do echo $i;  sed -i.bak '/FileName/d; /SourceFile/d; /FileType/d; /FileTypeExtension/d; /MIMEType/d; /MajorBrand/d; /Directory/d; /ExifToolVersion/d; /FileSize/d; /FileModifyDate/d; /MediaDataSize/d; /MediaDataOffset/d;  ' "${i}"; done;

## Импорт exif-данных из json-файлов в изображения 
<pre>
MAX_JOBS=16; \
NEWDIR="jsonexif/"; \
mkdir -pv "$NEWDIR"; \
time ( for i in *.*; do ( echo "   $i"; \
exiftool -overwrite_original -j+="jsonexif/${i%.*}.json" "${i}";  \
) & \
echo "Current jobs: $( jobs -r -p | wc -l )"; \
if [$(jobs -r -p | wc -l) -ge $(( $MAX_JOBS - 1 )) ]([)]; then wait -n; fi; \
done; wait; \
echo "All jobs done"; );
</pre>

## Написать некоторые exif данные текстом поверх каждой фотографии 
NEWDIR="GPStext"; mkdir -pv $NEWDIR;  time for i in *.*; do echo $i; { echo -e "\ndigital-artist@Borodin-Atamanov.ru"; exiftool -veryShort "${i}" | grep -E 'ExposureTime|SubSecModifyDate|GPSAltitude|GPSLatitude|GPSLongitude'; } | grep -v -E 'Ref|GPSAltitudeRef'| magick convert  "$i" -auto-orient -channel rgba -gravity north-west -strokewidth 1 -pointsize 77 -font Helvetica -background '#ABC0' -stroke '#000F' -fill '#FFFF'  label:@-   -composite -verbose -monitor -quality 100 -sampling-factor 4:4:4 "${NEWDIR}/${i}"; done;

# policy.xml 
*magick -list policy
*/etc/ImageMagick-6/policy.xml

<pre>
<nowiki>

<policymap xmlns="">
<policy domain="resource" name="temporary-path" value="/tmp"/>
<policy domain="resource" name="memory" value="4GiB"/>
<policy domain="resource" name="map" value="4GiB"/>
<policy domain="resource" name="width" value="10KP"/>
<policy domain="resource" name="height" value="10KP"/>
<policy domain="resource" name="list-length" value="128"/>
<policy domain="resource" name="area" value="100MP"/>
<policy domain="resource" name="disk" value="16EiB"/>
<policy domain="resource" name="file" value="768"/>
<policy domain="resource" name="thread" value="16"/>
<policy domain="resource" name="throttle" value="0"/>
<policy domain="resource" name="time" value="3600"/>
<policy domain="coder" rights="none" pattern="MVG" />
<policy domain="delegate" rights="none" pattern="HTTPS" />
<policy domain="path" rights="none" pattern="@*" />
<policy domain="cache" name="memory-map" value="anonymous"/>
<policy domain="cache" name="max-memory-request" value="256MiB"/>
<policy domain="cache" name="synchronize" value="True"/>
<policy domain="cache" name="shared-secret" value="passphrase" stealth="true"/>
<policy domain="system" name="shred" value="2"/>
<policy domain="system" name="precision" value="6"/>

 <policy domain="coder" rights="read|write" pattern="{HTTP,HTTPS,MVG,PS,EPS,PDF,XPS}" />
</policymap>
</nowiki></pre>

# Сохранить все списки ImageMagick в текстовые файлы 
<nowiki>
DIR="ImageMagick.list"; time for entity in `magick -list list`; do echo $entity; mkdir -p "$DIR"; magick -list "$entity" >"$DIR/$entity.txt"; done;
</nowiki>

# См. также 
*[подготовить фотографии для социальных сетей]([Как)]


[[[Категория:Linux]([Категория:Как]])]
[[[Категория:Convert]([Категория:imagemagick]])]
[[[Категория:Обработка изображений]([Категория:Изображения]])]
[[[Категория:Пакетная обработка]([Категория:Bash]])]
[[[Категория:Convert]([Категория:Ubuntu]])]
[[[Категория:Фото]([Категория:Консоль]])]
[[[Категория:Изображения]([Категория:Фотографии]])]
[изображений]([Категория:Обработка)]
[сети]([Категория:Социальные)]
[[[Категория:WWW]([Категория:Web]])]
[[[Категория:Jpeg]([Категория:jpg]])]
[[[Категория:Codecs]([Категория:Codec]])]
[[[Категория:Конвертирование]([Категория:Конвертер]])]
[Yi]([Категория:Xiaomi)]
[[[Категория:Дисторсия]([Категория:Barrel]])]
[Debarrel]([Категория:)]
[Убрать искажение]([Категория:)]
[Imagemagick]([Категория:)]

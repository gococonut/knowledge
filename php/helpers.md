### Image Helper

```php
<?php
namespace helpers;

class ImageHelper
{
    public static function autowrap($fontsize, $angle, $font, $string, $width, $wordSpace = null)
    {
        $content = '';
        $text = [];
        $count = mb_strlen($string);
        for ($i = 0; $i < $count; $i++) {
            $l = mb_substr($string, $i, 1);
            $content .= $l;
            $wordWidth = static::getTextWidth($font, $fontsize, $content, $wordSpace);
            if (($wordWidth > $width) && ($content !== '')) {
                $text[] = $content;
                $content = '';
            } else {
                if ($i == $count - 1) {
                    $text[] = $content;
                }
            }
        }

        return $text;
    }

    public static function imageTtfTextCenterHorizontally(&$image, $y, $font, $fontSize, $col, $text, $wordSpace = null)
    {
        $tipsTitleBox = imagettfbbox($fontSize, 0, $font, $text);
        $textWidth = ($tipsTitleBox[2] - $tipsTitleBox[0]);
        $count = mb_strlen($text);
        $textWidth += ($count - 1) * $wordSpace;
        $x = ceil((imagesx($image) - $textWidth) / 2);

        return static::imageTtfText(
            $image, $font, $fontSize, $col,
            $x, $y, $text, $wordSpace
        );
    }

    public static function centerHorizontallyTextArray(&$image, $y, $textArray, $wordSpace = null)
    {
        $textTotalWidth = 0;
        foreach ($textArray as $text) {
            $tipsTitleBox = imagettfbbox($text['fontSize'], 0, $text['font'], $text['value']);
            $textWidth = ($tipsTitleBox[2] - $tipsTitleBox[0]);
            $count = mb_strlen($text['value']);
            $textWidth += ($count - 1) * $wordSpace;
            $textTotalWidth += $textWidth;
        }

        $startX = ceil((imagesx($image) - $textTotalWidth) / 2);
        foreach ($textArray as $text) {
            $tipsTitleBox = imagettfbbox($text['fontSize'], 0, $text['font'], $text['value']);
            $textWidth = ($tipsTitleBox[2] - $tipsTitleBox[0]);
            $count = mb_strlen($text['value']);
            $textWidth += ($count - 1) * $wordSpace;
            $textY = $y;
            if (isset($text['yOffset'])) {
                $textY = $y + $text['yOffset'];
            }

            if (isset($text['xOffset'])) {
                $startX = $startX + $text['xOffset'];
            }

            static::imageTtfText(
                $image, $text['font'], $text['fontSize'], $text['col'],
                $startX, $textY, $text['value'], $wordSpace
            );

            $startX += $wordSpace + $textWidth;
        }

        return $image;
    }

    public static function getTextWidth($font, $fontSize, $text, $wordSpace = null)
    {
        $textBox = imagettfbbox($fontSize, 0, $font, $text);
        $textWidth = ($textBox[2] - $textBox[0]);
        $count = mb_strlen($text);
        $textWidth += ($count - 1) * $wordSpace;

        return $textWidth;
    }

    public static function getFontHeight($font, $fontSize, $char)
    {
        $charBox = imagettfbbox($fontSize, 0, $font, $char);
        $charHeight = ($charBox[3] - $charBox[5]);

        return $charHeight;
    }

    public static function imageTtfText(
        &$image, $font, $fontSize, $col,
         $x, $y, $text, $wordSpace = null
    ) {
        if (!isset($wordSpace)) {
            imagettftext($image, $fontSize, 0, $x, $y, $col, $font, $text);

            return $image;
        }

        $count = mb_strlen($text);
        for ($i = 0; $i < $count; $i++) {
            $t = mb_substr($text, $i, 1);
            imagettftext($image, $fontSize, 0, $x, $y, $col, $font, $t);
            $tipsTitleBox = imagettfbbox($fontSize, 0, $font, $t);
            $textWidth = ($tipsTitleBox[2] - $tipsTitleBox[0]);

            $x += $wordSpace + $textWidth;
        }

        return true;
    }

    public static function imageCreate($filePath)
    {
        if (static::isUrl($filePath)) {
            return imagecreatefromstring(file_get_contents($filePath));
        }

        $type = exif_imagetype($filePath);
        $allowedTypes = [
            1,  // gif
            2,  // jpg
            3,  // png
            6   // bmp
        ];
        if (!in_array($type, $allowedTypes)) {
            return false;
        }

        switch ($type) {
            case 1:
                $image = @imageCreateFromGif($filePath);
            break;
            case 2:
                $image = @imageCreateFromJpeg($filePath);
            break;
            case 3:
                $image = @imageCreateFromPng($filePath);
            break;
            case 6:
                $image = @imageCreateFromBmp($filePath);
            break;
        }

        return $image;
    }

    public static function imageTtfTextWithStartAndEndX(
        &$image, $font, $fontSize, $col,
         $startX, $endX, $y, $text, $lineWidth = null, $wordSpace = null
    ) {
        $fontHeight = static::getFontHeight($font, $fontSize, '国');
        $y += $fontHeight;
        $text = static::autowrap($fontSize, 0, $font, $text, $endX - $startX, $wordSpace);

        foreach ($text as $item) {
            static::imageTtfText($image, $font, $fontSize, $col, $startX, $y, $item, $wordSpace);
            $y = $y + $fontHeight;
            if (isset($lineWidth)) {
                $y += $lineWidth;
            }
        }

        return $y - $lineWidth;
    }

    public static function imageRoundedRectangle(&$image, $x, $y, $x2, $y2, $rad, $col)
    {
        imagefilledrectangle($image, $x, $y + $rad, $x2, $y2 - $rad, $col);
        imagefilledrectangle($image, $x + $rad, $y, $x2 - $rad, $y2, $col);

        $height = $rad * 2;

        imagefilledellipse($image, $x + $rad, $y + $rad, $rad * 2, $height, $col);
        imagefilledellipse($image, $x + $rad, $y2 - $rad, $rad * 2, $height, $col);
        imagefilledellipse($image, $x2 - $rad, $y2 - $rad, $rad * 2, $height, $col);
        imagefilledellipse($image, $x2 - $rad, $y + $rad, $rad * 2, $height, $col);
    }

    public static function isUrl($url)
    {
        return (bool)preg_match('/^(ftp|http|https):\/\/([\w-]+\.)+(\w+)(:[0-9]+)?(\/|([\w#!:.?+=&%@!\-\/]+))?$/', $url);
    }
}
```
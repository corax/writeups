# Writeup of Rainbow Hex

In the stego-challenge `Rainbow Hex` we got an mp4 file that quickly flashed some colorful images.

We used ffmpeg to extract the images from the files into a few hundred files, using this

`ffmpeg -i video.mp4 thumb%04d.png -hide_banner`

Looking at the files, we recognized it as [hexahue](https://www.geocachingtoolbox.com/index.php?lang=en&page=hexahue), where the pattern of the six colored squares indicates a character.

Next we wrote a python script that looped over all the .png files, grabbed the colors, translated from hexahue into ascii, and printed the text.


```py
#!/usr/bin/python3

from PIL import Image
import glob

# Script for parsing Hexahue images
# Expects all *.png files in current directory to be hexahue images.
# Written for "Rainbow Hex" challenge on ZH3R0 CTF
# Scalpel, Team Corax, june 2020

def hexahue_to_ascii(colors):
    """
    P = Purple
    R = Red
    G = Green
    Y = Yellow
    B = Blue
    C = Cyan
    b = black
    w = white
    g = gray/grey
    """

    hexahue = {
        "PRGYBC": "a",
        "RPGYBC": "b",
        "RGPYBC": "c",                                                                                                                                  
        "RGYPBC": "d",                                                                                                                                  
        "RGYBPC": "e",                                                                                                                                  
        "RGYBCP": "f",                                                                                                                                  
        "GRYBCP": "g",                                                                                                                                  
        "GYRBCP": "h",                                                                                                                                  
        "GYBRCP": "i",                                                                                                                                  
        "GYBCRP": "j",                                                                                                                                  
        "GYBCPR": "k",                                                                                                                                  
        "YGBCPR": "l",                                                                                                                                  
        "YBGCPR": "m",                                                                                                                                  
        "YBCGPR": "n",                                                                                                                                  
        "YBCPGR": "o",                                                                                                                                  
        "YBCPRG": "p",                                                                                                                                  
        "BYCPRG": "q",                                                                                                                                  
        "BCYPRG": "r",                                                                                                                                  
        "BCPYRG": "s",                                                                                                                                  
        "BCPRYG": "t",                                                                                                                                  
        "BCPRGY": "u",                                                                                                                                  
        "CBPRGY": "v",                                                                                                                                  
        "CPBRGY": "w",                                                                                                                                  
        "CPRBGY": "x",                                                                                                                                  
        "CPRGBY": "y",                                                                                                                                  
        "CPRGYB": "z",                                                                                                                                  
        "bwwbbw": ".",
        "wbbwwb": ",",
        "wwwwww": " ",
        "bbbbbb": " ",
        "bgwbgw": "0",
        "gbwbgw": "1",
        "gwbbgw": "2",
        "gwbgbw": "3",
        "gwbgwb": "4",
        "wgbgwb": "5",
        "wbggwb": "6",
        "wbgwgb": "7",
        "wbgwbg": "8",
        "bwgwbg": "9"
    }

    if colors in hexahue:
        return hexahue.get(colors)
    else:
        return '?'

def getHexahueFromFile(filename):
    img = Image.open(filename)
    pixels = img.load()

    # Pick the six colors from the image
    width, height = img.size
    first = pixels[10, 10]
    second = pixels[width-10, 10]
    third = pixels[width/4, height/2]
    fourth = pixels[width*3/4, height/2]
    fifth = pixels[10, height-10]
    sixth = pixels[width-10, height-10]

    colors = [first, second, third, fourth, fifth, sixth]

    #print(colors)

    hexahue = ''
    for (red,green,blue) in colors:

        if red > 240 and green > 240 and blue < 100:
            hexahue += 'Y'
        elif red < 100 and green < 100 and blue > 240:
            hexahue += 'B'
        elif 50 < red < 170 and green > 240 and blue < 100:
            hexahue += 'G'
        elif red < 100 and green > 240 and blue > 240: 
            hexahue += 'C'
        elif red > 240 and green < 100 and blue > 240:
            hexahue += 'P'
        elif red > 240 and green < 100 and blue < 100:
            hexahue += 'R'
        elif red < 100 and green < 100 and blue < 100:
            hexahue += 'b'
        elif red > 230 and green > 230 and blue > 230:
            hexahue += 'w'
        elif 50 < red < 150 and 50 < green < 150 and 50 < blue < 150:
            hexahue += 'g'
        else:
            hexahue += '?'

    # Any one we haven't solved?
    if '?' in hexahue:
        print(filename + " => " + str(colors) + " => " + hexahue )

    #print(hexahue)
    return hexahue


if __name__ == '__main__':

    # Loop over all images
    files = glob.glob("*.png")
    for filename in sorted(files): #, reverse=True):
        #print(filename)
        print(hexahue_to_ascii(getHexahueFromFile(filename)), end='')
```

Running the script prints out the text:

```
mr. robot follows elliot, a young programmer who works as a cyber security engineer by day and as a vigilante hacker by night. elliot finds himself at a crossroads when the mysterious leader of an underground hacker group recruits him to destroy the firm he is paid to protect. mr. robot had this hidden..start..zh3r0 w4s 1t 4 dr34m 0r was 1t 7h3 r34l1ty..end..compelled by his personal beliefs, elliot struggles to resist the chance to take down the multinational ceos he believes are running the world. eventually, he realizes that a global conspiracy does exist, but not the one he expected.
```
and from the middle section we get the flag `zh3r0{w4s_1t_4_dr34m_0r_was_1t_7h3_r34l1ty}`
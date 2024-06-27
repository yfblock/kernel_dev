If you want to use `xrandr` to scale kernel, the key need to note >

```shell
# set display scale, default scale. This is support for scale.
export QT_AUTO_SCREEN_SCALE_FACTOR=2
export QT_SCALE_FACTOR=2
export GDK_SCALE=2
export GDK_DPI_SCALE=1
# scale monitor, scale-from is like scale 2x2
xrandr --dpi 192  --output HDMI-A-0 --scale-from 3840x2160 --pos 3840x0
```
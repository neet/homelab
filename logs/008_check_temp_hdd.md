# HDDの温度を調べる

## やったこと

`smartmontools` をインストール

```
sudo apt install smartmontools
```

温度を測る

```
neet@mitaka-nas:~$ sudo smartctl -a /dev/sda | grep -i Temperature_Celsius
194 Temperature_Celsius     0x0022   025   040   000    Old_age   Always       -       25 (0 20 0 0 0)
neet@mitaka-nas:~$ sudo smartctl -a /dev/sdb | grep -i Temperature_Celsius
194 Temperature_Celsius     0x0022   025   040   000    Old_age   Always       -       25 (0 18 0 0 0)
neet@mitaka-nas:~$ sudo smartctl -a /dev/sdc | grep -i Temperature_Celsius
194 Temperature_Celsius     0x0022   025   040   000    Old_age   Always       -       25 (0 18 0 0 0)
```

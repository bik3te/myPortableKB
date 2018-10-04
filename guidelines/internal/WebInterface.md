# Web interfaces enumeration and pwning
## Quickly inspect web interfaces
We don't want to manually inspect web interfaces... Let's grab our Nmap XML report(s) and use [Paparazzi](https://github.com/bik3te/Paparazzi) to take a screenshot of every web interfaces:
```
$ python paparazzi.py 192.168.0_Quick.xml
```
Similar tools:
- [webscreenshot](https://github.com/maaaaz/webscreenshot)
- [httpscreenshot](https://github.com/breenmachine/httpscreenshot)
- [EyeWitness](https://github.com/FortyNorthSecurity/EyeWitness)


## Pwning common web interfaces
* [Tomcat](guidelines/internal/Tomcat.md)
* [JBoss](guidelines/internal/Tomcat.md)
* [Jenkins](guidelines/internal/Jenkins.md)

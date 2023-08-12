# digital-js

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).

# Digital display

## Format de base

<STX>NLXXXXXXXX<LF>

- STX = 0x02
- N = numéro de ligne <1..9, A..K> (total 1 ... 20)
- L = luminosité <1..3>
- X = caractères (jusqu’à 64)
- LF = 0x0A

Format: 8bits / no parity / 1 stop bit
Baud Rate: 9600bds

Tous les caractères standard ASCII <32 .. 126> à l’exception du caractère ‘^’ qui est utilisé
comme délimiteur
 !"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ[\]_'`abcdefghijklmnopqrstuvwxyz{|}~ 

Extension caractères latin ASCII (ISO-8859-1) <224 .. 255>
àáâãäåæçèéêëìíîïðñòóôõö÷øùúûüýþÿ 



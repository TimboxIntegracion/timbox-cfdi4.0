# CDFI 4.0

Documentación para la integración del Anexo 20 versión 4.0 (CFDI).

Aviso:

El 1 de enero de 2022 entra en vigor la versión 4.0 de la factura, no obstante podrás continuar emitiendo facturas en la versión 3.3 hasta el 30 de junio; a partir del 1 de julio, la única versión válida para emitir las facturas será la versión 4.0

[Documentación del SAT para emitir CFDI con la nueva versión 4.0](http://omawww.sat.gob.mx/tramitesyservicios/Paginas/anexo_20_version3-3.htm)

Ejemplo de cómo utilizar el servicio con [SOAP UI](https://www.soapui.org/downloads/soapui.html). Utilizamos la versión open source.

Se deberá hacer uso de la URL que hace referencia al WSDL, en cada petición realizada:

Timbrado:

- [Timbox Pruebas](https://staging.ws.timbox.com.mx/timbrado_cfdi40/wsdl)
- [Timbox Producción](https://sistema.timbox.com.mx/timbrado_cfdi40/wsdl)

Cancelación:
- [Timbox Pruebas](https://staging.ws.timbox.com.mx/cancelacion/wsdl)
- [Timbox Producción](https://sistema.timbox.com.mx/cancelacion/wsdl)


## Pasos a considerar antes de Timbrar CFDI
Para poder timbrar el CFDI, hay que considerar los siguientes pasos:

**<b>Paso 1</b>**
   
   Construir el XML en base al Anexo 20 de acuerdo al estándar definido por el SAT:

- [Esquema XSD](http://www.sat.gob.mx/sitio_internet/cfd/4/cfdv40.xsd)

- [Estándar PDF](http://omawww.sat.gob.mx/tramitesyservicios/Paginas/documentos/Anexo20_2022.pdf)


**<b>Paso 1.1 Obtener número de certificado</b>**
Si no se cuenta con el número de certificado se puede obtener utilizando un comando de OpenSSL:
  ```
openssl x509 -inform DER -in "CSD_INNOVACION_VALOR_Y_DESARROLLO_SA_DE_CV_IVD920810GU2_20190617_133410s.cer" -noout -serial
  ```
Ejemplo de output:
  ```
serial=3330303031303030303030343030303032343338
  ```
Es importante mencionar que este no es el número de certificado, para obtenerlo se debe eliminar el '3' en cada posición impar de la cadena de 40 caracteres. Ejemplo (hay espacio entre los pares para poder visualizar facilmente):
  ```
33 30 30 30 31 30 30 30 30 30 30 34 30 30 30 30 32 34 33 38
 3  0  0  0  1  0  0  0  0  0  0  4  0  0  0  0  2  4  3  8
 30001000000400002438
  ```
En este ejemplo el número de certificado sería: 30001000000400002438

**<b>Paso 2</b>** 
   
   Obtener la cadena Original basandose en el estándar XSLT (Secuencia de cadena Original), para realizar este paso utilizamos la siguiente herramienta  [XSL Test Tool](http://xslttest.appspot.com/) donde subiremos el archivo Estándar de tranformación y el XML.
   
   [Estándar de transformación](http://www.sat.gob.mx/sitio_internet/cfd/4/cadenaoriginal_4_0/cadenaoriginal_4_0.xslt)
 
 Ejemplo de una Cadena Original

  ```
||4.0|VG|11814|2022-05-30T11:58:16|01|30001000000400002438|2572.95|MXN|1|2725.59|I|01|PUE|52080|IVD920810GU2|INNOVACION VALOR Y DESARROLLO SA|601|XAXX010101000|PUBLICO EN GENERAL|52080|616|S01|10191509|PU03180194|12.000000|KGM|PRESTO 5 KILO (PRODUCTOS SANIDAD URBANA)|159.0000|1908.00|02|1908.00|002|Tasa|0.080000|152.64|10191509|FM03130005|1.000000|H87|QUICK FUME 500 PASTILLAS (FUMIGANTE)|664.9500|664.95|02|664.95|002|Tasa|0.000000|0.00|1908.00|002|Tasa|0.080000|152.64|664.95|002|Tasa|0.000000|0.00|152.64||
  ```

**<b>Paso 3</b>**
   
Generar sello digital para los CFDIs
Tal cómo lo específica el anexo 20 en el inciso I Sección B "Generación de sellos digitales para comprobantes fiscales digitales a través de Internet". Para esto usamos la siguiente herramienta, el programa [OPENSSL](https://www.openssl.org/) en el cual utilizamos los siguientes comandos de consola:

1. Obtener el PEM del certificado y el contenido sin los encabezados agregarlo al atributo Certificado en XML.
```
openssl x509 -in "CSD_INNOVACION_VALOR_Y_DESARROLLO_SA_DE_CV_IVD920810GU2_20190617_133410s.cer" -inform DER -out 'IVD920810GU2.cer.pem' -outform PEM
```
2. Obtener el PEM de la llave, con el cual se realizara la firma digital de la cadena original
```
openssl pkcs8 -inform DER -in "CSD_INNOVACION_VALOR_Y_DESARROLLO_SA_DE_CV_IVD920810GU2_20190617_133410.key" -passin pass:12345678a -out 'IVD920810GU2.key.pem'
```
3. Generación de la digestión o hash.
```
openssl dgst -sha256 -sign 'IVD920810GU2.key.pem' -out 'digest.txt' 'cadena_original.txt'
```

4. Creación del archivo PEM de la llave privada.
```
openssl enc -in 'digest.txt' -out 'sello.txt' -base64 -A -K 'IVD920810GU2.key.pem' 
```
Ejemplo de la Generación del Sello

```
ZLT7Y591WNw7f0hpWXDc1rAxs3K8K7QWh+yhnD1IUMkJegBhVBTBVJHDApLcFkNHj4e9XPu4YdNCvgPskSmY/MI8oueE9+Q3HbkbOwSAGe/SoHwcOQP/O4asuLel07yE3yo8KiZyqfpc0gGb4xCS3E3YGoAZcuOuz/2PUevr4iagI3lwy0qRnCQC7oFJa5kkeVJpZxsY/sPTCb6iCeGTyZr+Y/DoUQAtbndPsNWtAitjZr04HGkCxEPr793oz3ekGhw0qhk6Uao70814u4/dwH2z/GhfCksYW8iDayHF9m9ZZY35wHEanivm1vTKFCXoacG4uJKepmN2kvvC4KrvOw==
```


## Creación del proyecto en SOAP UI
Para iniciar con el ejemplo del timbrado es necesario crear el proyecto con el URL Servicio.

1. El primer paso es crear el proyecto.

    ![](http://i.imgur.com/0ar7zY0.png)

2. Lo siguiente es introducir los datos para generar el servicio, en Initial WSDL colocamos el URL que utilizaremos en este caso staging, debemos de asegurarnos de que este seleccionado los siguientes puntos:
    * **Create sample requests for all operations?**
    * **Stores all file paths in project relatively to project file (requires save)**
    
    Después presionaremos el botón de OK

     ![](http://i.imgur.com/fn6qM7N.png)
     
3. El siguiente paso es aceptar el directorio donde se guardará el proyecto.

     ![](http://i.imgur.com/UCq1NwS.png)
     
4. A continuación, nos mostrara el proyecto creado con las peticiones para cada uno de los métodos del servicio.

     ![](http://i.imgur.com/250CyFV.png)
     

## Timbrar CFDI

Para hacer una petición de timbrado de un CFDI, deberá enviar las credenciales asignadas, así cómo el XML que desea timbrar convertido a una cadena base64, para ello recomendamos utilizar la página [https://www.base64encode.org/](https://www.base64encode.org/) en ella se puede pegar el XML deseado y se obtiene la cadena en base64:

Para hacer la petición solo necesitamos hacer doble click sobre **Request 1** debajo de **timbrar_cfdi**:

     ![](http://i.imgur.com/wxkGZ25.png)
     
Después de dar click nos aparecerá la siguiente ventana, debemos modificar la petición usando nuestros datos, cómo el siguiente código:
![](http://i.imgur.com/YeoGMB6.png)
```
<soapenv:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:urn="urn:WashOut">
    <soapenv:Header/>
    <soapenv:Body>
      <urn:timbrar_cfdi soapenv:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
        <username xsi:type="xsd:string">usuario</username>
        <password xsi:type="xsd:string">constraseña</password>
        <sxml xsi:type="xsd:string">PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0iVVRGLTgiPz4KPGNmZGk6Q29tcHJvYmFudGUgeG1sbnM6Y2ZkaT0iaHR0cDovL3d3dy5zYXQuZ29iLm14L2NmZC80IiB4bWxuczp4c2k9Imh0dHA6Ly93d3cudzMub3JnLzIwMDEvWE1MU2NoZW1hLWluc3RhbmNlIiB4c2k6c2NoZW1hTG9jYXRpb249Imh0dHA6Ly93d3cuc2F0LmdvYi5teC9jZmQvNCBodHRwOi8vd3d3LnNhdC5nb2IubXgvc2l0aW9faW50ZXJuZXQvY2ZkLzQvY2ZkdjQwLnhzZCIgVmVyc2lvbj0iNC4wIiBTZXJpZT0iVkciIEZvbGlvPSIxMTgxNCIgRmVjaGE9IjIwMjItMDUtMzBUMTE6NTg6MTYiIEZvcm1hUGFnbz0iMDEiIE5vQ2VydGlmaWNhZG89IjMwMDAxMDAwMDAwNDAwMDAyNDM4IiBTdWJUb3RhbD0iMjU3Mi45NSIgTW9uZWRhPSJNWE4iIFRpcG9DYW1iaW89IjEiIFRvdGFsPSIyNzI1LjU5IiBUaXBvRGVDb21wcm9iYW50ZT0iSSIgRXhwb3J0YWNpb249IjAxIiBNZXRvZG9QYWdvPSJQVUUiIEx1Z2FyRXhwZWRpY2lvbj0iNTIwODAiIFNlbGxvPSJaTFQ3WTU5MVdOdzdmMGhwV1hEYzFyQXhzM0s4SzdRV2greWhuRDFJVU1rSmVnQmhWQlRCVkpIREFwTGNGa05IajRlOVhQdTRZZE5DdmdQc2tTbVkvTUk4b3VlRTkrUTNIYmtiT3dTQUdlL1NvSHdjT1FQL080YXN1TGVsMDd5RTN5bzhLaVp5cWZwYzBnR2I0eENTM0UzWUdvQVpjdU91ei8yUFVldnI0aWFnSTNsd3kwcVJuQ1FDN29GSmE1a2tlVkpwWnhzWS9zUFRDYjZpQ2VHVHlacitZL0RvVVFBdGJuZFBzTld0QWl0alpyMDRIR2tDeEVQcjc5M296M2VrR2h3MHFoazZVYW83MDgxNHU0L2R3SDJ6L0doZkNrc1lXOGlEYXlIRjltOVpaWTM1d0hFYW5pdm0xdlRLRkNYb2FjRzR1SktlcG1OMmt2dkM0S3J2T3c9PSIgQ2VydGlmaWNhZG89Ik1JSUYwakNDQTdxZ0F3SUJBZ0lVTXpBd01ERXdNREF3TURBME1EQXdNREkwTXpnd0RRWUpLb1pJaHZjTkFRRUxCUUF3Z2dFck1ROHdEUVlEVlFRRERBWkJReUJWUVZReExqQXNCZ05WQkFvTUpWTkZVbFpKUTBsUElFUkZJRUZFVFVsT1NWTlVVa0ZEU1U5T0lGUlNTVUpWVkVGU1NVRXhHakFZQmdOVkJBc01FVk5CVkMxSlJWTWdRWFYwYUc5eWFYUjVNU2d3SmdZSktvWklodmNOQVFrQkZobHZjMk5oY2k1dFlYSjBhVzVsZWtCellYUXVaMjlpTG0xNE1SMHdHd1lEVlFRSkRCUXpjbUVnWTJWeWNtRmtZU0JrWlNCallXUnBlakVPTUF3R0ExVUVFUXdGTURZek56QXhDekFKQmdOVkJBWVRBazFZTVJrd0Z3WURWUVFJREJCRFNWVkVRVVFnUkVVZ1RVVllTVU5QTVJFd0R3WURWUVFIREFoRFQxbFBRVU5CVGpFUk1BOEdBMVVFTFJNSU1pNDFMalF1TkRVeEpUQWpCZ2txaGtpRzl3MEJDUUlURm5KbGMzQnZibk5oWW14bE9pQkJRMFJOUVMxVFFWUXdIaGNOTVRrd05qRTNNakF3TkRFeVdoY05Nak13TmpFM01qQXdOREV5V2pDQitURXBNQ2NHQTFVRUF4TWdTVTVPVDFaQlEwbFBUaUJXUVV4UFVpQlpJRVJGVTBGU1VrOU1URThnVTBFeEtUQW5CZ05WQkNrVElFbE9UazlXUVVOSlQwNGdWa0ZNVDFJZ1dTQkVSVk5CVWxKUFRFeFBJRk5CTVNrd0p3WURWUVFLRXlCSlRrNVBWa0ZEU1U5T0lGWkJURTlTSUZrZ1JFVlRRVkpTVDB4TVR5QlRRVEVsTUNNR0ExVUVMUk1jU1ZaRU9USXdPREV3UjFVeUlDOGdTMEZJVHpZME1URXdNVUl6T1RFZU1Cd0dBMVVFQlJNVklDOGdTMEZJVHpZME1URXdNVWhPVkV4TFV6QTJNUzh3TFFZRFZRUUxGQ1pKVGs1UFZrRkRTZE5PSUZaQlRFOVNJRmtnUkVWVFFWSlNUMHhNVHlCVFFTQkVSU0JEVmpDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBSUtuM1U0ek1JRUdmQUlDc0RDU0Q0RVF3UWJhYldKT2RRcFhkbkFlQ3F3SUgvdkFibnY0dDdSYzFORk1KZzJPZUNLNUw2ZytQai9EbFdmdE9lcVJvOEtJelcwNHh0cTZyV1N4WFh1aEdEbTloZFJGMTVKRW94Sm5wRG1rcUtMRXBiTEpxWjRtT2RLT0hkbUVkUTRPM2RkMG5qZU5HZGpoVTUrVmw0SjlySFZqQTRCb0pwc1A3NUJRZUl3RGRGSVZ4Y2I2MHNLbmEwbUc1OGxNcTV0Tks2Q3RvemV5ckM5NElFRDlHQVJHMGdLdVhsc3dUcldySTFOMy9WT0k3ampaWTZZZUllTzVqY2JYckNpcUd4R2R1SnJHSHF5cnhHUFRJWGxaN2lXOHpNVDFHOFFSRjVyV3V0ZzQrTXlzSXNqcmNGcmV4YlA1VE1hUkJZeUFTZUJkOTRrQ0F3RUFBYU1kTUJzd0RBWURWUjBUQVFIL0JBSXdBREFMQmdOVkhROEVCQU1DQnNBd0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dJQkFKVnFnZFErTk9TSUJLYldCQlNBUm1YNUMwd2lxMEJmUno0VWI3ZFZHZ3VIb1BBQjRkSkF0TDU0MGIzS2xpZDkwRXd0TlNDQ0ZTYzlPUFEwbVdWUmtDejBPdGNlTEtBWHpLcXpWT2NYcnhteWRsYlQwYnJOQmtVTERWNTRHZHVwMUI0Mm1tQ2gwaG9SYU52cjBRa0puS0VNVG1IdDNtNDd4blY5WGZPZVVjU0RjaUNUTVY0d1JuczNKZENyN2cwUE15V3BWNXlSTEpOcjh3OVlDbTZZZXNTcUNKTFpJdldiRkdSdURoSGZnc3lJbXRNdVROYmNsYkk3WEZ1ZTJCNGF3aTNhOUg4eWZnZjMzYkdnZUxoVkNBYklMZDYzTFF2Njd6R2R0M0g4czJYZTR1bVFtaCs5Z3lKRzRmbWJabVVkM3dGbC9mWTlrbWRUeFpmdWZxUGJkeVYvRzdnMEhxM21telMxVHVSRzJIait5NGxWekxPVjZSWEpqSVErUXArNWVQS2w0ZC9vZUFJc1R6bVdzWmMwYyt5SjFNTWJDbjZDSFRsTnZkd1JPT2hwYW5OakVISjJrTHdoNW5ob1ZTUmJSRi9vd3VCcmVPNUZiSktaRStMKzhtellFSWhLVWlEMDNXVzAzQk9QQ0NRRkZ2QkZDMEFvTjd3UzB6Rm0xZzJnTlY5NXBnWndwMUxzcWx3QlNBVkY5U1MvTUtpRTdNdjBVcnlXRUJZbGxvNjZYWThWTUZ6WllBalN1RXVkb0szeFNzQ2RqYUEvV2p4ODUvM1JDRVYzSjdMTzVtNm13WWdyUUlhVHd1WVd1Q1RVRThYS3hqN2NqTm8xVUY2TGJMNzduemJZbVMzc1VYRTRKbHk4eHhPWXN5RFg4Qis4VE5XKytnMW1CM2J0K09IdiI+CiAgPGNmZGk6RW1pc29yIFJmYz0iSVZEOTIwODEwR1UyIiBOb21icmU9IklOTk9WQUNJT04gVkFMT1IgWSBERVNBUlJPTExPIFNBIiBSZWdpbWVuRmlzY2FsPSI2MDEiLz4KICA8Y2ZkaTpSZWNlcHRvciBSZmM9IlhBWFgwMTAxMDEwMDAiIE5vbWJyZT0iUFVCTElDTyBFTiBHRU5FUkFMIiBEb21pY2lsaW9GaXNjYWxSZWNlcHRvcj0iNTIwODAiIFJlZ2ltZW5GaXNjYWxSZWNlcHRvcj0iNjE2IiBVc29DRkRJPSJTMDEiLz4KICA8Y2ZkaTpDb25jZXB0b3M+CiAgICA8Y2ZkaTpDb25jZXB0byBDbGF2ZVByb2RTZXJ2PSIxMDE5MTUwOSIgTm9JZGVudGlmaWNhY2lvbj0iUFUwMzE4MDE5NCIgQ2FudGlkYWQ9IjEyLjAwMDAwMCIgQ2xhdmVVbmlkYWQ9IktHTSIgRGVzY3JpcGNpb249IlBSRVNUTyA1IEtJTE8gKFBST0RVQ1RPUyBTQU5JREFEIFVSQkFOQSkiIFZhbG9yVW5pdGFyaW89IjE1OS4wMDAwIiBJbXBvcnRlPSIxOTA4LjAwIiBPYmpldG9JbXA9IjAyIj4KICAgICAgPGNmZGk6SW1wdWVzdG9zPgogICAgICAgIDxjZmRpOlRyYXNsYWRvcz4KICAgICAgICAgIDxjZmRpOlRyYXNsYWRvIEJhc2U9IjE5MDguMDAiIEltcHVlc3RvPSIwMDIiIFRpcG9GYWN0b3I9IlRhc2EiIFRhc2FPQ3VvdGE9IjAuMDgwMDAwIiBJbXBvcnRlPSIxNTIuNjQiLz4KICAgICAgICA8L2NmZGk6VHJhc2xhZG9zPgogICAgICA8L2NmZGk6SW1wdWVzdG9zPgogICAgPC9jZmRpOkNvbmNlcHRvPgogICAgPGNmZGk6Q29uY2VwdG8gQ2xhdmVQcm9kU2Vydj0iMTAxOTE1MDkiIE5vSWRlbnRpZmljYWNpb249IkZNMDMxMzAwMDUiIENhbnRpZGFkPSIxLjAwMDAwMCIgQ2xhdmVVbmlkYWQ9Ikg4NyIgRGVzY3JpcGNpb249IlFVSUNLIEZVTUUgNTAwIFBBU1RJTExBUyAoRlVNSUdBTlRFKSIgVmFsb3JVbml0YXJpbz0iNjY0Ljk1MDAiIEltcG9ydGU9IjY2NC45NSIgT2JqZXRvSW1wPSIwMiI+CiAgICAgIDxjZmRpOkltcHVlc3Rvcz4KICAgICAgICA8Y2ZkaTpUcmFzbGFkb3M+CiAgICAgICAgICA8Y2ZkaTpUcmFzbGFkbyBCYXNlPSI2NjQuOTUiIEltcHVlc3RvPSIwMDIiIFRpcG9GYWN0b3I9IlRhc2EiIFRhc2FPQ3VvdGE9IjAuMDAwMDAwIiBJbXBvcnRlPSIwLjAwIi8+CiAgICAgICAgPC9jZmRpOlRyYXNsYWRvcz4KICAgICAgPC9jZmRpOkltcHVlc3Rvcz4KICAgIDwvY2ZkaTpDb25jZXB0bz4KICA8L2NmZGk6Q29uY2VwdG9zPgogIDxjZmRpOkltcHVlc3RvcyBUb3RhbEltcHVlc3Rvc1RyYXNsYWRhZG9zPSIxNTIuNjQiPgogICAgPGNmZGk6VHJhc2xhZG9zPgogICAgICA8Y2ZkaTpUcmFzbGFkbyBCYXNlPSIxOTA4LjAwIiBJbXB1ZXN0bz0iMDAyIiBUaXBvRmFjdG9yPSJUYXNhIiBUYXNhT0N1b3RhPSIwLjA4MDAwMCIgSW1wb3J0ZT0iMTUyLjY0Ii8+CiAgICAgIDxjZmRpOlRyYXNsYWRvIEJhc2U9IjY2NC45NSIgSW1wdWVzdG89IjAwMiIgVGlwb0ZhY3Rvcj0iVGFzYSIgVGFzYU9DdW90YT0iMC4wMDAwMDAiIEltcG9ydGU9IjAuMDAiLz4KICAgIDwvY2ZkaTpUcmFzbGFkb3M+CiAgPC9jZmRpOkltcHVlc3Rvcz4KPC9jZmRpOkNvbXByb2JhbnRlPgo=</sxml>
    </urn:timbrar_cfdi>
    </soapenv:Body>
  </soapenv:Envelope>
```
Después daremos click al botón ![](http://i.imgur.com/zp9cg7E.png) una vez hecho esto nos saldrá el resultado.

  ![](http://i.imgur.com/fwG4Rc2.png)


## Cancelar CFDI 
Para la cancelar_cfdi son necesarias las credenciales asignadas, RFC del emisor, un arreglo de nodos folios (el cual debe contener los elementos UUID, RFC del Receptor, Total, Motivo y Folio_Sustituto), el certificado y llave convertidos en PEM (el contenido del archivo)

A partir del 2022 será necesario señalar el motivo de la cancelación de los comprobantes. Al seleccionar como motivo de cancelación la clave 01 “Comprobante emitido con errores con relación deberá relacionarse el folio fiscal del comprobante que sustituye al cancelado. Se actualizan los plazos para realizar la cancelación de facturas.

**<b> Motivos de Cancelación (Código - Descripción) </b>**
**<b>  01    -    Comprobante emitido con errores con relación </b>**
**<b>  02    -    Comprobante emitido con errores sin relación </b>**
**<b>  03    -    No se llevó a cabo la operación </b>**
**<b>  04    -    Operación nominativa relacionada en la factura global </b>**

Crear un cliente para hacer la petición de cancelación al webservice:

Para hacer la petición solo necesitamos hacer doble click sobre **Request 1** debajo de **cancelar_cfdi**:
![](https://i.imgur.com/RVyGwDm.png)

```
<soapenv:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:urn="urn:WashOut">
<soapenv:Header/>
<soapenv:Body>
   <urn:cancelar_cfdi soapenv:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
      <username xsi:type="xsd:string">usuario</username>
      <password xsi:type="xsd:string">contraseña</password>
      <rfc_emisor xsi:type="xsd:string">IVD920810GU2</rfc_emisor>
      <folios xsi:type="urn:folios">
         <!--Zero or more repetitions:-->
         <folio xsi:type="urn:folio">
            <uuid xsi:type="xsd:string">CF741B8A-A398-412E-BC97-6D1AE4B10069</uuid>
            <rfc_receptor xsi:type="xsd:string">IAD121214B34</rfc_receptor>
            <total xsi:type="xsd:string">7261.60</total>
            <motivo xsi:type="xsd:string">01</motivo>
            <folio_sustituto xsi:type="xsd:string">8D4B79A4-B17A-4B4B-9220-9225F73B8945</folio_sustituto>
         </folio>
      </folios>
      <cert_pem xsi:type="xsd:string">-----BEGIN CERTIFICATE-----
MIIF0jCCA7qgAwIBAgIUMzAwMDEwMDAwMDA0MDAwMDI0MzgwDQYJKoZIhvcNAQEL
BQAwggErMQ8wDQYDVQQDDAZBQyBVQVQxLjAsBgNVBAoMJVNFUlZJQ0lPIERFIEFE
TUlOSVNUUkFDSU9OIFRSSUJVVEFSSUExGjAYBgNVBAsMEVNBVC1JRVMgQXV0aG9y
aXR5MSgwJgYJKoZIhvcNAQkBFhlvc2Nhci5tYXJ0aW5lekBzYXQuZ29iLm14MR0w
GwYDVQQJDBQzcmEgY2VycmFkYSBkZSBjYWRpejEOMAwGA1UEEQwFMDYzNzAxCzAJ
BgNVBAYTAk1YMRkwFwYDVQQIDBBDSVVEQUQgREUgTUVYSUNPMREwDwYDVQQHDAhD
T1lPQUNBTjERMA8GA1UELRMIMi41LjQuNDUxJTAjBgkqhkiG9w0BCQITFnJlc3Bv
bnNhYmxlOiBBQ0RNQS1TQVQwHhcNMTkwNjE3MjAwNDEyWhcNMjMwNjE3MjAwNDEy
WjCB+TEpMCcGA1UEAxMgSU5OT1ZBQ0lPTiBWQUxPUiBZIERFU0FSUk9MTE8gU0Ex
KTAnBgNVBCkTIElOTk9WQUNJT04gVkFMT1IgWSBERVNBUlJPTExPIFNBMSkwJwYD
VQQKEyBJTk5PVkFDSU9OIFZBTE9SIFkgREVTQVJST0xMTyBTQTElMCMGA1UELRMc
SVZEOTIwODEwR1UyIC8gS0FITzY0MTEwMUIzOTEeMBwGA1UEBRMVIC8gS0FITzY0
MTEwMUhOVExLUzA2MS8wLQYDVQQLFCZJTk5PVkFDSdNOIFZBTE9SIFkgREVTQVJS
T0xMTyBTQSBERSBDVjCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAIKn
3U4zMIEGfAICsDCSD4EQwQbabWJOdQpXdnAeCqwIH/vAbnv4t7Rc1NFMJg2OeCK5
L6g+Pj/DlWftOeqRo8KIzW04xtq6rWSxXXuhGDm9hdRF15JEoxJnpDmkqKLEpbLJ
qZ4mOdKOHdmEdQ4O3dd0njeNGdjhU5+Vl4J9rHVjA4BoJpsP75BQeIwDdFIVxcb6
0sKna0mG58lMq5tNK6CtozeyrC94IED9GARG0gKuXlswTrWrI1N3/VOI7jjZY6Ye
IeO5jcbXrCiqGxGduJrGHqyrxGPTIXlZ7iW8zMT1G8QRF5rWutg4+MysIsjrcFre
xbP5TMaRBYyASeBd94kCAwEAAaMdMBswDAYDVR0TAQH/BAIwADALBgNVHQ8EBAMC
BsAwDQYJKoZIhvcNAQELBQADggIBAJVqgdQ+NOSIBKbWBBSARmX5C0wiq0BfRz4U
b7dVGguHoPAB4dJAtL540b3Klid90EwtNSCCFSc9OPQ0mWVRkCz0OtceLKAXzKqz
VOcXrxmydlbT0brNBkULDV54Gdup1B42mmCh0hoRaNvr0QkJnKEMTmHt3m47xnV9
XfOeUcSDciCTMV4wRns3JdCr7g0PMyWpV5yRLJNr8w9YCm6YesSqCJLZIvWbFGRu
DhHfgsyImtMuTNbclbI7XFue2B4awi3a9H8yfgf33bGgeLhVCAbILd63LQv67zGd
t3H8s2Xe4umQmh+9gyJG4fmbZmUd3wFl/fY9kmdTxZfufqPbdyV/G7g0Hq3mmzS1
TuRG2Hj+y4lVzLOV6RXJjIQ+Qp+5ePKl4d/oeAIsTzmWsZc0c+yJ1MMbCn6CHTlN
vdwROOhpanNjEHJ2kLwh5nhoVSRbRF/owuBreO5FbJKZE+L+8mzYEIhKUiD03WW0
3BOPCCQFFvBFC0AoN7wS0zFm1g2gNV95pgZwp1LsqlwBSAVF9SS/MKiE7Mv0UryW
EBYllo66XY8VMFzZYAjSuEudoK3xSsCdjaA/Wjx85/3RCEV3J7LO5m6mwYgrQIaT
wuYWuCTUE8XKxj7cjNo1UF6LbL77nzbYmS3sUXE4Jly8xxOYsyDX8B+8TNW++g1m
B3bt+OHv
-----END CERTIFICATE-----
</cert_pem>
      <llave_pem xsi:type="xsd:string">-----BEGIN PRIVATE KEY-----
MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQCCp91OMzCBBnwC
ArAwkg+BEMEG2m1iTnUKV3ZwHgqsCB/7wG57+Le0XNTRTCYNjngiuS+oPj4/w5Vn
7TnqkaPCiM1tOMbauq1ksV17oRg5vYXURdeSRKMSZ6Q5pKiixKWyyameJjnSjh3Z
hHUODt3XdJ43jRnY4VOflZeCfax1YwOAaCabD++QUHiMA3RSFcXG+tLCp2tJhufJ
TKubTSugraM3sqwveCBA/RgERtICrl5bME61qyNTd/1TiO442WOmHiHjuY3G16wo
qhsRnbiaxh6sq8Rj0yF5We4lvMzE9RvEERea1rrYOPjMrCLI63Ba3sWz+UzGkQWM
gEngXfeJAgMBAAECggEAFiEQXppU8MWEY2LJLLDQZ2/LAbolJK1dLW865CpybEjE
AgPJsr2hf67pbLmVCF7FAjyTUc+ZA3vA5mVLless7Vn2UTV4mLtdetx/lNzoGX98
F0PtCx0M8aUUL58v4MGlvu5hCCQ5Tuw7KghBOyxRbpiV45rGcfFYFINlsfhPKWJp
vt5YND3x/L2fH7q8iqjnixesbUUdwg7+0D9Y00ROgokFEOpHg5k+c9q2n4P+RNPI
CUpGcJJqM8/bFDhKgsu09k/TMVAQc2eyLIwNwuPUB9FyF1bBgH45G4IgQ6gaYnwA
ndGmqF4TycNjQ/GODlxbHJmWioknQL/qqZ/hS/v+AQKBgQDA88mN7XpSK/XZ8Hzj
t4lzCyLrefdicIYLMhpwrMf4JTOWN5Oi1WkYaXTUNhRh3XzgGr5Ky5AZq/srnepG
8iIy2JsUhxSOr1ZsH/P1qem72GcwEH2Eg5cugReQYoJz5cl+Tq2Cpq9XtxuF35dd
P12SVyx9sIviitXf3wSr05HcWQKBgQCtWQxl4TI1LWFzCIZfxSKkvv99oiMA1zuy
0VuI0xhlDGp4Jyi2jubahU8C5pViMj7xLF+SMax0nZGxmpq3QcZq96cuLW2yWaV4
BonP0cGqYtsPP9EOK1V3T9MF5bjHJVG8ac4fSRnqMLmEGbZZ+R+vINT86s2XwxWR
ZIuCeD3OsQKBgA17OxLqi8hf/+55ShCTC0x5c7gmLm23VPZFSumieNpSpxcQzQTs
ikpFW/9Tw/rOgeIanD8Xl/rjNEpo3yyT0GXjEnrNsVcC0zP8y4vXklgol5UZIdv3
YcHDDUVuTJUSchCcKK1fPhMP3SFubOH8AmquIpKpmix67NSWfXoP7zoJAoGBAIg0
Hb+3MCIEZDtkiWCantvfjxQB34r7ktawFUHuy44qMUXzTtQSeGVetXRMBThAzp/l
A7r0+NIwNJfeKI6xSdwmdt+bpkOqmI80Y/g8kfT087aJqBOADQlQWTibBZLESfLH
F8QRRiFy43FeWp9bVX/fRjrrq1sBV+MDo3KCU94hAoGAfTfcg497BUvtveKiFZJO
+0YTpN0KdWQJc81rh14x4mGoo575gxYN2pPOWLVsA6JRyNktYvjT3jgiQmBiG09x
jPuQbp6EVgZ6ivBzKXw0+4K9htjOR/WxQqwE5SrhrYwYLDJc734ve2tVXhR3YQf0
NJ3R5CwogC3cxHR20ETAxrE=
-----END PRIVATE KEY-----
</llave_pem>
   </urn:cancelar_cfdi>
</soapenv:Body>
```

Después daremos click al botón ![](http://i.imgur.com/zp9cg7E.png) y una vez hecho esto nos saldrá el resultado.

![](https://i.imgur.com/ipYKkUf.png)

### Consultar Estatus
El servicio de “consultar_estatus” se utiliza para la consulta del estatus del CFDI, este servicio pretende proveer una forma alternativa de consulta que requiera verificar el estado de un comprobante en bases de datos del SAT. 

Para utilizar el servicio son necesarias las credenciales asignadas, UUID, RFC del emisor, RFC del receptor y el Total.

Crear un cliente para hacer la petición de cancelación al webservice:

Para hacer la petición solo necesitamos hacer doble click sobre **Request 1** debajo de **consultar_estatus**:

```
<soapenv:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:urn="urn:WashOut">
   <soapenv:Header/>
   <soapenv:Body>
      <urn:consultar_estatus soapenv:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
         <username xsi:type="xsd:string">usuario</username>
         <password xsi:type="xsd:string">contraseña</password>
         <uuid xsi:type="xsd:string">6B0D9A22-CE8E-4F00-BFD4-4DD294D6A772</uuid>
         <rfc_emisor xsi:type="xsd:string">PZA000413788</rfc_emisor>
         <rfc_receptor xsi:type="xsd:string">TME960709LR2</rfc_receptor>
         <total xsi:type="xsd:string">5001</total>
      </urn:consultar_estatus>
   </soapenv:Body>
</soapenv:Envelope>
```

Después daremos click al botón ![](http://i.imgur.com/zp9cg7E.png) y una vez hecho esto nos saldrá el resultado.

![](https://i.imgur.com/SNy0sWd.png)


### Consultar Peticiones Pendientes
El servicio de “consultar_peticiones_pendientes” se utiliza para realizar la consulta al servicio del SAT para revisar las peticiones que se encuentran en espera de una respuesta por parte del Receptor

Para utilizar el servicio son necesarias las credenciales asignadas, RFC del receptor, el certificado y llave convertidos en PEM (el contenido del archivo)

Crear un cliente para hacer la petición de cancelación al webservice:

Para hacer la petición solo necesitamos hacer doble click sobre **Request 1** debajo de **consultar_peticiones_pendientes**:

![](https://i.imgur.com/8U0w4yf.png)

```
<soapenv:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:urn="urn:WashOut">
   <soapenv:Header/>
   <soapenv:Body>
      <urn:consultar_peticiones_pendientes soapenv:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
         <username xsi:type="xsd:string">usuario</username>
         <password xsi:type="xsd:string">contraseña</password>
         <rfc_receptor xsi:type="xsd:string">TME960709LR2</rfc_receptor>
         <cert_pem xsi:type="xsd:string">-----BEGIN CERTIFICATE-----
MIIFzDCCA7SgAwIBAgIUMjAwMDEwMDAwMDAzMDAwMjI3NjMwDQYJKoZIhvcNAQEL
BQAwggFmMSAwHgYDVQQDDBdBLkMuIDIgZGUgcHJ1ZWJhcyg0MDk2KTEvMC0GA1UE
CgwmU2VydmljaW8gZGUgQWRtaW5pc3RyYWNpw7NuIFRyaWJ1dGFyaWExODA2BgNV
BAsML0FkbWluaXN0cmFjacOzbiBkZSBTZWd1cmlkYWQgZGUgbGEgSW5mb3JtYWNp
w7NuMSkwJwYJKoZIhvcNAQkBFhphc2lzbmV0QHBydWViYXMuc2F0LmdvYi5teDEm
MCQGA1UECQwdQXYuIEhpZGFsZ28gNzcsIENvbC4gR3VlcnJlcm8xDjAMBgNVBBEM
BTA2MzAwMQswCQYDVQQGEwJNWDEZMBcGA1UECAwQRGlzdHJpdG8gRmVkZXJhbDES
MBAGA1UEBwwJQ295b2Fjw6FuMRUwEwYDVQQtEwxTQVQ5NzA3MDFOTjMxITAfBgkq
hkiG9w0BCQIMElJlc3BvbnNhYmxlOiBBQ0RNQTAeFw0xNjEwMjEyMDU0MDFaFw0y
MDEwMjEyMDU0MDFaMIG4MRwwGgYDVQQDExNJTk1PQiBFRE1BIFNBIERFIENWMRww
GgYDVQQpExNJTk1PQiBFRE1BIFNBIERFIENWMRwwGgYDVQQKExNJTk1PQiBFRE1B
IFNBIERFIENWMSUwIwYDVQQtExxUTUU5NjA3MDlMUjIgLyBIRUdUNzYxMDAzNFMy
MR4wHAYDVQQFExUgLyBIRUdUNzYxMDAzTURGUk5OMDkxFTATBgNVBAsUDFBydWVi
YXNfQ0ZESTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAJgqZ+ezJyeJ
XACMK8ehFp64ecAl8jfYKB4xMJy0RRb+qXKRewxtLojiTFECWdCx283tEkdHUj8b
LzsCfFAMnsP2G4CS2aE2/1LLCHoZpdImaasLX1YJL2bUzxKQKi+RlL63M49yyfvG
BjEgG7f6TMwVSUSbgDFpYAFHqx4LK+p2GVHuUUzoiIm8xRYaW1YPMa457be5W8ws
jw0nGRLfo8hRIjPHedkwtcqYPj57xsPXMfxWP45vOlW7GuLkMq/ECccHxJiPitiT
hcDFKlf/mAR0kaux9LTffvWilA2uQAlyVyNVjqfdpvDdq4ycTaoIMYKrv/9R31dQ
0AmdXT8cfbcCAwEAAaMdMBswDAYDVR0TAQH/BAIwADALBgNVHQ8EBAMCBsAwDQYJ
KoZIhvcNAQELBQADggIBAF5kwvyBUp7Ad99DktzEhrJwnMQyhA79sVc4Ns2SpLON
/cV244ZnG5hgXk2awKbHEiSj/ke7EhgEpGS818ERsj7eW/wRgugBZraVn48GOn6q
X0uV9EjwWEGK5uT6IDN25igeXxVJHP3hn40fX2BPqsaqRP49YMxcOWD7mhWRh2E6
BnoKYjgHVJbavUN6pjCBLmy4hKwfitbjqtUiiWOmBDvvmLFpEGXG8OXn2xladBUk
fC4sfgMBpVZVuEV7RqAgCSCZ2xo6UEyd4KKpTjbdp0Tj5gw+NmiovAZHwU/NPRoj
N95f/ibj7268LBr2DcO5rlmr7szwJ3dtwu86N7HkUxW3vo3qGHTVK2HRBArda9VN
4pEyIL0Qt46ci5rFYXB2cCWU8XAh8gaZnxJoNTSY4A4yMJG9UfM/2rHC+YvOouIZ
2kJZ2h+SwKOYGJOX749P/QeF4Z/L/ODs3E08bV7IQna1ZHmd6ydYhZVpheMgNoNn
IG6jdzfyuo8NZAIIW/JGmPTANPCwTSHqBY1lmnp/oZNrkxGWtGhbltRfBoFQfTqC
ZALm6fsVeQqHQ6a7W45FJ2RD1nltPSdniMo3Iz/t4eHCjFvM3aORvA9oJEPr5Zzz
BV2fQOXkyS8QdsSVb5ZmJG+FqZKKlsiaX6xhqK6gqTLyJN+7/yr9T/ZZ4M7VrRoL
-----END CERTIFICATE-----</cert_pem>
         <llave_pem xsi:type="xsd:string">-----BEGIN PRIVATE KEY-----
MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQCYKmfnsycniVwA
jCvHoRaeuHnAJfI32CgeMTCctEUW/qlykXsMbS6I4kxRAlnQsdvN7RJHR1I/Gy87
AnxQDJ7D9huAktmhNv9Sywh6GaXSJmmrC19WCS9m1M8SkCovkZS+tzOPcsn7xgYx
IBu3+kzMFUlEm4AxaWABR6seCyvqdhlR7lFM6IiJvMUWGltWDzGuOe23uVvMLI8N
JxkS36PIUSIzx3nZMLXKmD4+e8bD1zH8Vj+ObzpVuxri5DKvxAnHB8SYj4rYk4XA
xSpX/5gEdJGrsfS03371opQNrkAJclcjVY6n3abw3auMnE2qCDGCq7//Ud9XUNAJ
nV0/HH23AgMBAAECggEAYxh6wnHxtdXGjLS8bi2CRatt3qzXqXaj6cWvGt5rgCYo
w+vqbpVMEOkPOlKFm1u5Acq6dKEF9wMFJzDfNGKDoqrMDleUU2E1tf1zb9D0JH/P
oQyu8aDZteYxVK1+S6xLakh0056127mCnsuCQbZH/UB/jqaWPZeaZjr+PXqZBv8O
NDMzomZaIKSUqwwiFi/vSunrrbGEkcCjrLK7mtHVNOx4HuuGSwixIUbBPefbR2Po
2fIATTKTIGlAzZrKaVWW3730ZNO4WJeKvahoHrLiiFz9pCWr4h+WOSWQDE3MNrCr
LVIusjciKZ6bNkYK15p0SmQNh1A8ZCC/xx2zo0pHgQKBgQDmF+QgS6cWdwuDqus3
weShruOaIpzgp01h2+g5sl93Wbh7hXKgAP3MUDbdd/j8ZZ0R1hL2ChWw0t9Ma/29
o+sTSs+m4kTRiNf4qq6VoWlIdQ8CpEIEdBp5RC/Oxd1vNHSV0POQSM1v27lzOgkG
Aaq16vTCtm8LcztFE9Mys7Om+QKBgQCpTFyVKful82n3SLnhfVAypkUPcjHqsIZf
KSVLdDCdQJGRKdKvMcr/4A+xRC1SdUPE5RUEIq7aQIRe27H2JjD3USXPGJCC0pah
ct9QxZajqIsw4CpodOiTuXBphaIfjGibSCE83pKQbRfoKIhtuVHxaizLBjNGvapS
JYzG368GLwKBgQDH4dFHTPElztyt0PjtQv6+hhMqfw8RCcVrUYH3PUE5iTN9+nuN
C89ugfBnjCU7/XnpWLK4EiKtrUJWPSn8aD16UO765m0qKVqUppFrYwD29NnJTbAb
9lBZMCbn1XN7e3IcA5zSpqvwlEwSEURtd105E5b0306v/7ZpV8OMtBdI4QKBgGF/
38XsAshk8f7+/EYXdEtnJFir7IF7njdJq/fTd3foyqyuSG6rH3zTHlZ5rBxT+m53
e+4Ax3BcPZ+fqNLY1dRpAHxPalJdU3CxhlivIn0oQNkqEGJOCe+hmVK8Kk0/ALOF
C9dRW1kf6ufCCCgg1UdSXW+jJ36zFlbu1y9lfRfzAoGALViSE59U7+pOk324H5yh
rAotmRzydr94AAkM7b4KWT6Lm9KNO9jhJlpcyHMOqT+yvDzNSBcpCMh+bCFwNUCh
oqM8++PNBZhYg8uRr236Z2dwQLE8W3uvlGVkmld/w1oSss/IVGIW9Fkzq+FmXst6
nG6Qen20YfqezK2yWdYNDnU=
-----END PRIVATE KEY-----</llave_pem>
      </urn:consultar_peticiones_pendientes>
   </soapenv:Body>
</soapenv:Envelope>
```

Después daremos click al botón ![](http://i.imgur.com/zp9cg7E.png) y una vez hecho esto nos saldrá el resultado.

![](https://i.imgur.com/hsLPmJ1.png)


### Procesar Respuesta
El servicio de “procesar_respuesta” se utiliza para realizar la petición de aceptación/rechazo de la solicitud de cancelación que se encuentra en espera de dicha resolución por parte del receptor del documento al servicio del SAT.

Para utilizar el servicio son necesarias las credenciales asignadas, RFC del receptor, un arreglo de nodos respuestas (el cual debe contener los elementos UUID, RFC del Emisor, Total y Respuesta la cual puede ser "A" de Aceptacion y "R" de rechazo), el certificado y llave convertidos en PEM (el contenido del archivo)

Crear un cliente para hacer la petición de cancelación al webservice:

Para hacer la petición solo necesitamos hacer doble click sobre **Request 1** debajo de **procesar_respuesta**:

![](https://i.imgur.com/N0nFvXR.png)

```
<soapenv:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:urn="urn:WashOut">
   <soapenv:Header/>
   <soapenv:Body>
      <urn:procesar_respuesta soapenv:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
         <username xsi:type="xsd:string">usuario</username>
         <password xsi:type="xsd:string">contraseña</password>
         <rfc_receptor xsi:type="xsd:string">TME960709LR2</rfc_receptor>
         <respuestas xsi:type="urn:respuestas">
         		<folios_respuestas>
		         	<uuid xsi:type="xsd:string">F3D2B562-2DD0-4566-8517-E36F80AE5B46</uuid>
		         	<rfc_emisor xsi:type="xsd:string">PZA000413788</rfc_emisor>
		         	<total xsi:type="xsd:string">5001</total>
		         	<respuesta xsi:type="xsd:string">A</respuesta>
	         	</folios_respuestas>
         </respuestas>
         <cert_pem xsi:type="xsd:string">-----BEGIN CERTIFICATE-----
MIIFzDCCA7SgAwIBAgIUMjAwMDEwMDAwMDAzMDAwMjI3NjMwDQYJKoZIhvcNAQEL
BQAwggFmMSAwHgYDVQQDDBdBLkMuIDIgZGUgcHJ1ZWJhcyg0MDk2KTEvMC0GA1UE
CgwmU2VydmljaW8gZGUgQWRtaW5pc3RyYWNpw7NuIFRyaWJ1dGFyaWExODA2BgNV
BAsML0FkbWluaXN0cmFjacOzbiBkZSBTZWd1cmlkYWQgZGUgbGEgSW5mb3JtYWNp
w7NuMSkwJwYJKoZIhvcNAQkBFhphc2lzbmV0QHBydWViYXMuc2F0LmdvYi5teDEm
MCQGA1UECQwdQXYuIEhpZGFsZ28gNzcsIENvbC4gR3VlcnJlcm8xDjAMBgNVBBEM
BTA2MzAwMQswCQYDVQQGEwJNWDEZMBcGA1UECAwQRGlzdHJpdG8gRmVkZXJhbDES
MBAGA1UEBwwJQ295b2Fjw6FuMRUwEwYDVQQtEwxTQVQ5NzA3MDFOTjMxITAfBgkq
hkiG9w0BCQIMElJlc3BvbnNhYmxlOiBBQ0RNQTAeFw0xNjEwMjEyMDU0MDFaFw0y
MDEwMjEyMDU0MDFaMIG4MRwwGgYDVQQDExNJTk1PQiBFRE1BIFNBIERFIENWMRww
GgYDVQQpExNJTk1PQiBFRE1BIFNBIERFIENWMRwwGgYDVQQKExNJTk1PQiBFRE1B
IFNBIERFIENWMSUwIwYDVQQtExxUTUU5NjA3MDlMUjIgLyBIRUdUNzYxMDAzNFMy
MR4wHAYDVQQFExUgLyBIRUdUNzYxMDAzTURGUk5OMDkxFTATBgNVBAsUDFBydWVi
YXNfQ0ZESTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAJgqZ+ezJyeJ
XACMK8ehFp64ecAl8jfYKB4xMJy0RRb+qXKRewxtLojiTFECWdCx283tEkdHUj8b
LzsCfFAMnsP2G4CS2aE2/1LLCHoZpdImaasLX1YJL2bUzxKQKi+RlL63M49yyfvG
BjEgG7f6TMwVSUSbgDFpYAFHqx4LK+p2GVHuUUzoiIm8xRYaW1YPMa457be5W8ws
jw0nGRLfo8hRIjPHedkwtcqYPj57xsPXMfxWP45vOlW7GuLkMq/ECccHxJiPitiT
hcDFKlf/mAR0kaux9LTffvWilA2uQAlyVyNVjqfdpvDdq4ycTaoIMYKrv/9R31dQ
0AmdXT8cfbcCAwEAAaMdMBswDAYDVR0TAQH/BAIwADALBgNVHQ8EBAMCBsAwDQYJ
KoZIhvcNAQELBQADggIBAF5kwvyBUp7Ad99DktzEhrJwnMQyhA79sVc4Ns2SpLON
/cV244ZnG5hgXk2awKbHEiSj/ke7EhgEpGS818ERsj7eW/wRgugBZraVn48GOn6q
X0uV9EjwWEGK5uT6IDN25igeXxVJHP3hn40fX2BPqsaqRP49YMxcOWD7mhWRh2E6
BnoKYjgHVJbavUN6pjCBLmy4hKwfitbjqtUiiWOmBDvvmLFpEGXG8OXn2xladBUk
fC4sfgMBpVZVuEV7RqAgCSCZ2xo6UEyd4KKpTjbdp0Tj5gw+NmiovAZHwU/NPRoj
N95f/ibj7268LBr2DcO5rlmr7szwJ3dtwu86N7HkUxW3vo3qGHTVK2HRBArda9VN
4pEyIL0Qt46ci5rFYXB2cCWU8XAh8gaZnxJoNTSY4A4yMJG9UfM/2rHC+YvOouIZ
2kJZ2h+SwKOYGJOX749P/QeF4Z/L/ODs3E08bV7IQna1ZHmd6ydYhZVpheMgNoNn
IG6jdzfyuo8NZAIIW/JGmPTANPCwTSHqBY1lmnp/oZNrkxGWtGhbltRfBoFQfTqC
ZALm6fsVeQqHQ6a7W45FJ2RD1nltPSdniMo3Iz/t4eHCjFvM3aORvA9oJEPr5Zzz
BV2fQOXkyS8QdsSVb5ZmJG+FqZKKlsiaX6xhqK6gqTLyJN+7/yr9T/ZZ4M7VrRoL
-----END CERTIFICATE-----</cert_pem>
         <llave_pem xsi:type="xsd:string">-----BEGIN PRIVATE KEY-----
MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQCYKmfnsycniVwA
jCvHoRaeuHnAJfI32CgeMTCctEUW/qlykXsMbS6I4kxRAlnQsdvN7RJHR1I/Gy87
AnxQDJ7D9huAktmhNv9Sywh6GaXSJmmrC19WCS9m1M8SkCovkZS+tzOPcsn7xgYx
IBu3+kzMFUlEm4AxaWABR6seCyvqdhlR7lFM6IiJvMUWGltWDzGuOe23uVvMLI8N
JxkS36PIUSIzx3nZMLXKmD4+e8bD1zH8Vj+ObzpVuxri5DKvxAnHB8SYj4rYk4XA
xSpX/5gEdJGrsfS03371opQNrkAJclcjVY6n3abw3auMnE2qCDGCq7//Ud9XUNAJ
nV0/HH23AgMBAAECggEAYxh6wnHxtdXGjLS8bi2CRatt3qzXqXaj6cWvGt5rgCYo
w+vqbpVMEOkPOlKFm1u5Acq6dKEF9wMFJzDfNGKDoqrMDleUU2E1tf1zb9D0JH/P
oQyu8aDZteYxVK1+S6xLakh0056127mCnsuCQbZH/UB/jqaWPZeaZjr+PXqZBv8O
NDMzomZaIKSUqwwiFi/vSunrrbGEkcCjrLK7mtHVNOx4HuuGSwixIUbBPefbR2Po
2fIATTKTIGlAzZrKaVWW3730ZNO4WJeKvahoHrLiiFz9pCWr4h+WOSWQDE3MNrCr
LVIusjciKZ6bNkYK15p0SmQNh1A8ZCC/xx2zo0pHgQKBgQDmF+QgS6cWdwuDqus3
weShruOaIpzgp01h2+g5sl93Wbh7hXKgAP3MUDbdd/j8ZZ0R1hL2ChWw0t9Ma/29
o+sTSs+m4kTRiNf4qq6VoWlIdQ8CpEIEdBp5RC/Oxd1vNHSV0POQSM1v27lzOgkG
Aaq16vTCtm8LcztFE9Mys7Om+QKBgQCpTFyVKful82n3SLnhfVAypkUPcjHqsIZf
KSVLdDCdQJGRKdKvMcr/4A+xRC1SdUPE5RUEIq7aQIRe27H2JjD3USXPGJCC0pah
ct9QxZajqIsw4CpodOiTuXBphaIfjGibSCE83pKQbRfoKIhtuVHxaizLBjNGvapS
JYzG368GLwKBgQDH4dFHTPElztyt0PjtQv6+hhMqfw8RCcVrUYH3PUE5iTN9+nuN
C89ugfBnjCU7/XnpWLK4EiKtrUJWPSn8aD16UO765m0qKVqUppFrYwD29NnJTbAb
9lBZMCbn1XN7e3IcA5zSpqvwlEwSEURtd105E5b0306v/7ZpV8OMtBdI4QKBgGF/
38XsAshk8f7+/EYXdEtnJFir7IF7njdJq/fTd3foyqyuSG6rH3zTHlZ5rBxT+m53
e+4Ax3BcPZ+fqNLY1dRpAHxPalJdU3CxhlivIn0oQNkqEGJOCe+hmVK8Kk0/ALOF
C9dRW1kf6ufCCCgg1UdSXW+jJ36zFlbu1y9lfRfzAoGALViSE59U7+pOk324H5yh
rAotmRzydr94AAkM7b4KWT6Lm9KNO9jhJlpcyHMOqT+yvDzNSBcpCMh+bCFwNUCh
oqM8++PNBZhYg8uRr236Z2dwQLE8W3uvlGVkmld/w1oSss/IVGIW9Fkzq+FmXst6
nG6Qen20YfqezK2yWdYNDnU=
-----END PRIVATE KEY-----</llave_pem>
      </urn:procesar_respuesta>
   </soapenv:Body>
</soapenv:Envelope>
```

Después daremos click al botón ![](http://i.imgur.com/zp9cg7E.png) y una vez hecho esto nos saldrá el resultado.

![](https://i.imgur.com/bf34dv1.png)





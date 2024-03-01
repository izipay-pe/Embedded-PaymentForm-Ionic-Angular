# Embedded-PaymentForm-Ionic-Angular

## Índice

- [1. Introducción](#1-introducción)
- [2. Requisitos previos](#2-requisitos-previos)
- [3. Despliegue](#3-despliegue)
- [4. Datos de conexión](#4-datos-de-conexión)
- [5. Transacción de prueba](#5-transacción-de-prueba)
- [6. Implementación de la IPN](#6-implementación-de-la-ipn)
- [7. Personalización](#7-personalización)
- [8. Consideraciones](#8-consideraciones)

## 1. Introducción

En este manual podrás encontrar una guía paso a paso para configurar un proyecto de **[ Ionic - Angular ]** con la pasarela de pagos de IZIPAY. Te proporcionaremos instrucciones detalladas y credenciales de prueba para la instalación y configuración del proyecto, permitiéndote trabajar y experimentar de manera segura en tu propio entorno local.
Este manual está diseñado para ayudarte a comprender el flujo de la integración de la pasarela para ayudarte a aprovechar al máximo tu proyecto y facilitar tu experiencia de desarrollo.

<p align="center">
  <img src="https://raw.githubusercontent.com/izipay-pe/Imagenes/main/formulario_incrustado/Imagen-Formulario-Incrustado.png" alt="Formulario" width="350"/>
</p>

<a name="Requisitos_Previos"></a>

## 2. Requisitos previos

- Comprender el flujo de comunicación de la pasarela. [Información Aquí](https://secure.micuentaweb.pe/doc/es-PE/rest/V4.0/javascript/guide/start.html)
- Extraer credenciales del Back Office Vendedor. [Guía Aquí](https://github.com/izipay-pe/obtener-credenciales-de-conexion)
- Debe instalar la [versión de LTS node.js](https://nodejs.org/es/).
- Para este proyecto utilizamos la herramienta Visual Studio Code.
  > [!NOTE]
  > Tener en cuenta que, para que el desarrollo de tu proyecto, eres libre de emplear tus herramientas preferidas.

## 3. Despliegue

### Clonar el proyecto:

sh
git clone [        ]


### Ejecutar proyecto

- Ingrese a la carpeta raiz del proyecto.

- A continuación, instale el cliente ionic-cli:

  bash
     npm install -g @ionic/cli
  

  Más detalles en la página web de [ionic](https://ionicframework.com/docs/intro/cli).

- Agregue la dependencia con:

  bash
    npm install --save @lyracom/embedded-form-glue
  

Ejecútelo con el siguiente comando:

sh
ionic serve


ver el resultado en http://localhost:8100/

## 4. Datos de conexión

*Nota: Reemplace *[CHANGE_ME]** con sus credenciales de API REST extraídas desde el Back Office Vendedor, ver [Requisitos Previos](#Requisitos_Previos).

- Editar en src/index.html en la sección HEAD.

  ```javascript
  <!-- tema y plugins. debe cargarse en la sección HEAD -->
  <link rel="stylesheet"
  href="~~CHANGE_ME_ENDPOINT~~/static/js/krypton-client/V4.0/ext/classic-reset.css">
  <script
      src="~~CHANGE_ME_ENDPOINT~~/static/js/krypton-client/V4.0/ext/classic.js">
  </script>
  ``` 
  

- Edita src/app/app.component.html template, se agregará al elemento #myPaymentForm para ver formulario de pago.

 ```html
  <ion-content>
    <div class="form-container">
      <div id="myPaymentForm">
        <div class="kr-embedded"></div>
      </div>
      <div data-test="payment-message">{{message}}</div>
    </div>
    <p></p>
  </ion-content>
  ``` 

- Actualice los estilos en src/app/app.component.css.

  ```css
  .form-container {
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  ``` 
  

- Edite el componente predeterminado src/app/app.component.ts, con el siguiente codigo si quiere interactuar con el formulario de pago, con un endpoint propio.

    ``` js
  import { HttpClient } from '@angular/common/http';
  import { Component, AfterViewInit, ChangeDetectorRef } from '@angular/core';
  import KRGlue from '@lyracom/embedded-form-glue';
  @Component({
  selector: 'app-root',
  templateUrl: 'app.component.html',
  styleUrls: ['app.component.scss'],
  })
  export class AppComponent implements AfterViewInit {
    message = '';

    constructor(private http: HttpClient, private chRef: ChangeDetectorRef) {}

    ngAfterViewInit(){

      /*Colocar las credenciales extraídas del Back Office Vendedor para mas detalle regresar a:  Requisitos Previos. */

      const endpoint = "~~CHANGE_ME_ENDPOINT~~";
      const publicKey = "~~CHANGE_ME_PUBLIC_KEY~~";
      let formToken = ''; /* Variable se mantiene vacío */
     
     /* Abre una nueva conexión, utilizando la solicitud POST en el URL de su endpoint '*/
      const observable = this.http.post(
      'http://localhost:3000/createPayment',
       /* Arreglo que enviara la data al endpoint */
      { paymentConf: { amount: 10000, currency: 'PEN' } },
      { responseType: 'json' }
    )
     observable
      .toPromise()
      .then((resp: any) => {
        formToken = resp.formtoken; 
        /* cargar la libreria KRGlue */
        return KRGlue.loadLibrary(
          endpoint,
          publicKey
        ); 
      })
      .then(({ KR }) =>
         KR.setFormConfig({
           /* establecer la configuración mínima */
          formToken: formToken,
          /* cambia el idioma del formulario */
          'kr-language': 'es-PE', 
           })
         )
        .then(({ KR }) =>
         /*Renderiza el formulario de pago a myPaymentForm div */
        KR.renderElements('#myPaymentForm')
      )
     .catch(error => {
        this.message = error.message + ' (see console for more details)'
      })
    }
  }
  ``` 
  
## 5. Transacción de prueba

Antes de poner en marcha su pasarela de pago en un entorno de producción, es esencial realizar pruebas para garantizar su correcto funcionamiento.

Puede intentar realizar una transacción utilizando una tarjeta de prueba con la barra de herramientas de depuración (en la parte inferior de la página).

<p align="center">
  <img src="https://i.postimg.cc/3xXChGp2/tarjetas-prueba.png" alt="Formulario"/>
</p>

- También puede encontrar tarjetas de prueba en el siguiente enlace. [Tarjetas de prueba](https://secure.micuentaweb.pe/doc/es-PE/rest/V4.0/api/kb/test_cards.html)

## 6. Implementación de la IPN

> [!IMPORTANT]
> Es recomendable implementar la IPN para comunicar el resultado de la solicitud de pago al servidor del comercio.

La IPN es una notificación de servidor a servidor (servidor de Izipay hacia el servidor del comercio) que facilita información en tiempo real y de manera automática cuando se produce un evento, por ejemplo, al registrar una transacción.
Los datos transmitidos en la IPN se reciben y analizan mediante un script que el vendedor habrá desarrollado en su servidor.

- Ver manual de implementación de la IPN. [Aquí](https://secure.micuentaweb.pe/doc/es-PE/rest/V4.0/kb/payment_done.html)
- Vea el ejemplo de la respuesta IPN con PHP. [Aquí](https://github.com/izipay-pe/Redirect-PaymentForm-IpnT1-PHP)
- Vea el ejemplo de la respuesta IPN con NODE.JS. [Aquí](https://github.com/izipay-pe/Response-PaymentFormT1-Ipn)

## 7. Personalización

Si deseas aplicar cambios específicos en la apariencia de la pasarela de pago, puedes lograrlo mediante la modificación de código CSS. En este enlace [Código CSS - Incrustado](https://github.com/izipay-pe/Personalizacion-PaymentForm-Incrustado) podrá encontrar nuestro script para un formulario incrustado.

## 8. Consideraciones

Para obtener más información, echa un vistazo a:

- [Formulario incrustado: prueba rápida](https://secure.micuentaweb.pe/doc/es-PE/rest/V4.0/javascript/quick_start_js.html)
- [Primeros pasos: pago simple](https://secure.micuentaweb.pe/doc/es-PE/rest/V4.0/javascript/guide/start.html)
- [Servicios web - referencia de la API REST](https://secure.micuentaweb.pe/doc/es-PE/rest/V4.0/api/reference.html)

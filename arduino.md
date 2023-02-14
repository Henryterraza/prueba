``` c++
    #include <LedControl.h>

    #define TOTAL_LETRAS 16  //Ancho del mensaje total
    #define TAM_LETRAS 5     //Ancho de cada letra
    #define TAM_NUM 6        //Ancho de cada número
    #define IZQUIERDA 0
    #define DERECHA 1
    #define PIN_INI 23
    #define TOTAL_PIN 16

    #define DIN 5
    #define CLK 6
    #define LOAD 7

    #define BTN_IZQ 10
    #define BTN_INI 9
    #define BTN_DER 8

    #define SOUNDER 11


    LedControl matrizDriver = LedControl(DIN, CLK, LOAD, 1);

    unsigned int estadoMatriz[16] = { 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 };

    unsigned int matrizPausada[16] = { 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 };

    // Datos codificados de los símbolos a mostrar
    int letras[16][8] = {
    { 0, 0, 0, 6, 15, 217, 217, 3 },
    { 0, 0, 0, 255, 136, 136, 136, 248 },\ 
                        { 0, 0, 0, 255, 152, 148, 146, 241 },
    { 0, 0, 0, 255, 136, 136, 136, 255 },\ 
                        { 0, 0, 0, 1, 65, 255, 1, 1 },
    { 0, 0, 0, 36, 36, 36, 36, 36 },     \ 
                        { 0, 0, 0, 255, 129, 137, 137, 207 },
    { 0, 0, 0, 255, 152, 148, 146, 241 },\ 
                        { 0, 0, 0, 255, 1, 1, 1, 255 },
    { 0, 0, 0, 255, 136, 136, 136, 248 },\ 
                        { 0, 0, 0, 255, 129, 129, 129, 255 },
    { 0, 0, 0, 1, 65, 255, 1, 1 },       \ 
                        { 0, 0, 0, 1, 65, 255, 1, 1 },
    { 0, 0, 0, 36, 36, 36, 36, 36 },     \ 
                        { 0, 0, 0, 255, 136, 136, 136, 255 },
    { 0, 0, 0, 192, 155, 155, 240, 96 },
    };

    unsigned int numeros[10][8] = {
    { 0, 60, 102, 74, 82, 102, 60, 0 }, { 0, 0, 18, 34, 126, 2, 2, 0 }, { 0, 38, 74, 74, 74, 74, 50, 0 }, { 0, 68, 66, 66, 82, 114, 76, 0 }, { 0, 120, 8, 8, 8, 126, 0, 0 }, { 0, 114, 82, 82, 82, 82, 76, 0 }, { 0, 60, 74, 82, 82, 82, 76, 0 }, { 0, 64, 64, 70, 72, 80, 96, 0 }, { 0, 44, 82, 82, 82, 82, 44, 0 }, { 0, 48, 72, 72, 72, 72, 62, 0 }
    };

    // Organización de los símbolos del mensaje.
    int ordenMensaje[TOTAL_LETRAS] = { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15 };  //{¿PRA1=GRUPO11=A?}
    int estado_juego = 0;

    int x_ini_msg = -97;
    int direccionActual = DERECHA;
    int segundos = 0;
    bool btn_pressed = false;
    uint32_t tmp_inicio = 0;

    //Buzzer
    int frecuencia_buzzer = 128;

    //Barra y pelota
    int pos_barra = 6;
    int inicio_juego = 0;
    int dir_x = 1;
    int dir_y = 1;
    int pos_pelota_x = 8;
    int pos_pelota_y = 1;

    //Puntos y vidas
    int vidas = 3;
    int puntos = 0;
    int puntosAnteriores = 0;

    //Pausa / Configuración
    int inicio_pausa = 0;

    void mostrar(int opcion) {
    for (int p = 0; p <= 7; p++) {
        estadoMatriz[p] = numeros[(opcion / 10) % 10][p];
    }
    for (int p = 8; p <= 15; p++) {
        estadoMatriz[p] = numeros[opcion % 10][p];
    }
    }

    void mostrar_columnas() {

    int indice = 0;
    for (int i = PIN_INI + 8; i < PIN_INI + 16; i++) {
        int columna = estadoMatriz[indice];
        digitalWrite(i, HIGH);

        for (int j = PIN_INI; j < PIN_INI + 8; j++) {
        int pixel = columna & 1;
        columna = columna >> 1;
        if (pixel) {
            digitalWrite(j, LOW);
        }
        }

        delay(1);
        digitalWrite(i, LOW);

        for (int j = PIN_INI; j < PIN_INI + 8; j++) {
        digitalWrite(j, HIGH);
        }
        indice++;
    }
    }


    void actualizarMatriz() {
    for (int i = 0; i < 8; i++) {
        matrizDriver.setColumn(0, i, estadoMatriz[i + 8]);
    }
    }

    // Limpia la matriz y coloca todo el arreglo de estado en 0.
    void limpiarMensaje() {
    matrizDriver.clearDisplay(0);
    for (int i = 0; i < 16; i++) {
        estadoMatriz[i] = 0;
    }
    }

    // Coloca el mensaje en el arreglo de estado.
    void ponerMensaje() {

    int xActual = x_ini_msg;

    for (int i = 0; i < TOTAL_LETRAS; i++) {
        for (int j = 0; j < TAM_LETRAS; j++) {

        if (xActual >= 0 && xActual <= 15) {
            estadoMatriz[xActual] = letras[ordenMensaje[i]][j + 3];
        }
        xActual++;
        }
        xActual++;
    }
    }


    void recomenzar() {
    if (x_ini_msg == -97)  //para cuando sea de DER-IZQ
        x_ini_msg = 17;
    else if (x_ini_msg == 17)  //para cuando sea de IZQ-DER
        x_ini_msg = -97;
    }


    void moverMensaje(int direccion) {
    if (direccion == 0)
        x_ini_msg--;
    else if (direccion == 1)
        x_ini_msg++;
    }

    void mover_barra(int pos) {
    if (pos == 0) {
        if (pos_barra > 0) {
        pos_barra--;
        for (int i = 0; i < 16; i++) {
            if (i >= pos_barra && i <= pos_barra + 4) {
            bitSet(estadoMatriz[i], 0);
            } else {
            bitClear(estadoMatriz[i], 0);
            }
        }
        }
    } else if (pos == 1) {
        if (pos_barra < 11) {
        pos_barra++;
        for (int i = 0; i < 16; i++) {
            if (i >= pos_barra && i <= pos_barra + 4) {
            bitSet(estadoMatriz[i], 0);
            } else {
            bitClear(estadoMatriz[i], 0);
            }
        }
        }
    }

    delay(20);
    }

    void activar_buzzer(int time) {
    analogWrite(SOUNDER, frecuencia_buzzer);
    delay(time);
    analogWrite(SOUNDER, 0);
    }

    void verificarGanada() {
    if (puntos == 32) {
        limpiarMensaje();
        actualizarMatriz();
        activar_buzzer(1500);
        x_ini_msg = -97;
        direccionActual = DERECHA;
        estado_juego = 0;
        vidas = 3;
        puntos = 0;
        pos_pelota_x = 8;
        pos_pelota_y = 1;
        dir_x = 1;
        dir_y = 1;
        pos_barra = 6;
    }
    }

    void verificarVidaExtra() {
    if (puntos > puntosAnteriores && puntos % 2 == 0) {
        vidas++;
        puntosAnteriores = puntos;
    }
    }

    void verificarPerdida() {
    if (vidas == 0) {
        mostrar(puntos - 1);
        actualizarMatriz();
        for (int s = 0; s < 75; s++) {
        mostrar_columnas();
        delay(1);
        }
        x_ini_msg = -97;
        direccionActual = DERECHA;
        estado_juego = 0;
        vidas = 3;
        puntos = 0;
        inicio_juego = 0;
        pos_pelota_x = 8;
        pos_pelota_y = 1;
        dir_x = 1;
        dir_y = 1;
        pos_barra = 6;
    }
    }

    void mostrar_pelota() {
    bitClear(estadoMatriz[pos_pelota_x], pos_pelota_y);
    pos_pelota_x = pos_pelota_x + dir_x;
    pos_pelota_y = pos_pelota_y + dir_y;
    bitSet(estadoMatriz[pos_pelota_x], pos_pelota_y);
    }

    void mover_pelota() {

    if (pos_pelota_x == 15) {
        dir_x = -1;
        activar_buzzer(10);
    } else if (pos_pelota_x == 0) {
        dir_x = 1;
        activar_buzzer(10);
    }

    if (pos_pelota_y == 7) {
        dir_y = -1;
        activar_buzzer(10);
    } else if (pos_pelota_y == 0) {
        vidas--;
        verificarPerdida();
        bitClear(estadoMatriz[pos_pelota_x], pos_pelota_y);
        delay(250);
        pos_pelota_y = 1;
        pos_pelota_x = random(0, 15);
        dir_x = 1;
        dir_y = 1;
    }

    if (pos_pelota_y - 1 == 0 && bitRead(estadoMatriz[pos_pelota_x], 0) == 1) {
        dir_y = 1;
        activar_buzzer(10);
        delay(10);
    } else if (bitRead(estadoMatriz[pos_pelota_x], pos_pelota_y + dir_y) == 1 && bitRead(estadoMatriz[pos_pelota_x + dir_x], pos_pelota_y) == 0) {
        bitClear(estadoMatriz[pos_pelota_x], pos_pelota_y + dir_y);
        if (pos_pelota_x % 2 == 0) {
        bitClear(estadoMatriz[pos_pelota_x + 1], pos_pelota_y + dir_y);
        } else {
        bitClear(estadoMatriz[pos_pelota_x - 1], pos_pelota_y + dir_y);
        }
        puntos++;
        verificarVidaExtra();
        verificarGanada();
        if (dir_y == 1) {
        dir_y = -1;
        } else {
        dir_y = 1;
        }
        activar_buzzer(10);
    } else if (bitRead(estadoMatriz[pos_pelota_x], pos_pelota_y + dir_y) == 0 && bitRead(estadoMatriz[pos_pelota_x + dir_x], pos_pelota_y) == 1) {
        bitClear(estadoMatriz[pos_pelota_x + dir_x], pos_pelota_y);
        if ((pos_pelota_x + dir_x) % 2 == 0) {
        bitClear(estadoMatriz[pos_pelota_x + dir_x + 1], pos_pelota_y);
        } else {
        bitClear(estadoMatriz[pos_pelota_x + dir_x - 1], pos_pelota_y);
        }
        puntos++;
        verificarVidaExtra();
        verificarGanada();
        if (dir_x == 1) {
        dir_x = -1;
        } else {
        dir_x = 1;
        }
        activar_buzzer(10);
    } else if (bitRead(estadoMatriz[pos_pelota_x], pos_pelota_y + dir_y) == 1 && bitRead(estadoMatriz[pos_pelota_x + dir_x], pos_pelota_y) == 1) {
        bitClear(estadoMatriz[pos_pelota_x + dir_x], pos_pelota_y);
        bitClear(estadoMatriz[pos_pelota_x], pos_pelota_y + dir_y);
        if ((pos_pelota_x + dir_x) % 2 == 0) {
        bitClear(estadoMatriz[pos_pelota_x + dir_x + 1], pos_pelota_y);
        } else {
        bitClear(estadoMatriz[pos_pelota_x + dir_x - 1], pos_pelota_y);
        }
        if (pos_pelota_x % 2 == 0) {
        bitClear(estadoMatriz[pos_pelota_x + 1], pos_pelota_y + dir_y);
        } else {
        bitClear(estadoMatriz[pos_pelota_x - 1], pos_pelota_y + dir_y);
        }
        if (dir_x == 1) {
        dir_x = -1;
        } else {
        dir_x = 1;
        }
        if (dir_y == 1) {
        dir_y = -1;
        } else {
        dir_y = 1;
        }
        puntos++;
        puntos++;
        activar_buzzer(10);
        verificarVidaExtra();
        verificarGanada();
    }
    }


    void setup() {
    Serial.begin(9600);

    matrizDriver.shutdown(0, false);
    matrizDriver.setIntensity(0, 8);
    //potenciometro
    pinMode(A0, INPUT);

    //Entrada para la matriz 8x8
    for (int i = 23; i < TOTAL_PIN + PIN_INI + 1; i++) {
        pinMode(i, OUTPUT);
    }

    //  LOW para comlumnas
    for (int i = PIN_INI + (TOTAL_PIN / 2); i < TOTAL_PIN + PIN_INI; i++) {
        digitalWrite(i, LOW);
    }
    //HIGH para filas
    for (int i = PIN_INI; i < PIN_INI + TOTAL_PIN; i++) {
        digitalWrite(i, HIGH);
    }

    pinMode(BTN_IZQ, INPUT);
    pinMode(BTN_DER, INPUT);
    pinMode(BTN_INI, INPUT);

    pinMode(SOUNDER, OUTPUT);
    }


    void loop() {

    if (digitalRead(BTN_IZQ) == HIGH) {
        direccionActual = 1;
    } else if (digitalRead(BTN_DER) == HIGH) {
        direccionActual = 0;
    }


    if (estado_juego == 0) {
        int velocidad = map(analogRead(A0), 0, 1023, 5, 40);
        if (digitalRead(BTN_INI) == HIGH && btn_pressed) {
        uint32_t tmp_transcurrido = millis() - tmp_inicio;  //15 - 12 = 3
        if (tmp_transcurrido >= 1500) {                     //
            estado_juego = 1;
            btn_pressed = false;
            inicio_juego = 0;
        } else {
            estado_juego = 0;
        }
        } else if (digitalRead(BTN_INI) == HIGH) {  //
        btn_pressed = true;                       //false---true
        tmp_inicio = millis();                    //10
        } else {
        tmp_inicio = 0;
        btn_pressed = false;
        segundos = 0;
        }

        limpiarMensaje();
        actualizarMatriz();
        ponerMensaje();
        actualizarMatriz();
        for (int s = 0; s < velocidad; s++) {
        mostrar_columnas();
        delay(1);
        }
        moverMensaje(direccionActual);
        recomenzar();
    } else if (estado_juego == 1) {

        if (inicio_juego == 0) {
        limpiarMensaje();
        delay(1000);
        tmp_inicio = 0;
        btn_pressed = false;
        segundos = 0;
        for (int i = 0; i < 16; i++) {
            estadoMatriz[i] = 240;
            if (i > 5 && i < 11) {
            if (i == 8) {
                estadoMatriz[i] = 243;
            } else {
                estadoMatriz[i] = 241;
            }
            } else {
            estadoMatriz[i] = 240;
            }
        }
        }
        inicio_juego = 1;

        actualizarMatriz();
        for (int s = 0; s < 10; s++) {
        mostrar_columnas();
        delay(1);
        }

        for (int i = 0; i < 2; i++) {
        if (digitalRead(BTN_IZQ) == HIGH) {
            mover_barra(0);
        } else if (digitalRead(BTN_DER) == HIGH) {
            mover_barra(1);
        }
        mover_pelota();
        }
        delay(20);
        mostrar_pelota();

        if (digitalRead(BTN_INI) == HIGH && btn_pressed) {
        uint32_t tmp_transcurrido = millis() - tmp_inicio;  //15 - 12 = 3
        if (tmp_transcurrido >= 1500) {                     //
            estado_juego = 2;
            btn_pressed = false;
            tmp_inicio = 0;
            segundos = 0;
        }
        } else if (digitalRead(BTN_INI) == HIGH) {  //
        btn_pressed = true;                       //false---true
        tmp_inicio = millis();                    //10
        } else {
        tmp_inicio = 0;
        btn_pressed = false;
        segundos = 0;
        }


    } else if (estado_juego == 2) {
        //Estado de configuración

        if (inicio_pausa == 0) {
        for (int p = 0; p <= 15; p++) {
            matrizPausada[p] = estadoMatriz[p];
        }
        inicio_pausa = 1;
        }

        limpiarMensaje();
        mostrar(vidas - 1);
        actualizarMatriz();
        for (int s = 0; s < 10; s++) {
        mostrar_columnas();
        delay(1);
        }

        uint32_t tmp_transcurrido = millis() - tmp_inicio;
        if (digitalRead(BTN_INI) == HIGH && btn_pressed) {
        uint32_t tmp_transcurrido = millis() - tmp_inicio;  //15 - 12 = 3
        if(tmp_transcurrido >= 1500){
            limpiarMensaje();
            actualizarMatriz();
            for (int s = 0; s < 10; s++) {
            mostrar_columnas();
            delay(1);
            }
            x_ini_msg = -97;
            direccionActual = DERECHA;
            estado_juego = 0;
            tmp_transcurrido = 0;
            delay(500);
            vidas = 3;
            puntos = 0;
            inicio_juego = 0;
            pos_pelota_x = 8;
            pos_pelota_y = 1;
            dir_x = 1;
            dir_y = 1;
            pos_barra = 6;
        }else if(tmp_transcurrido >= 1000 ){
            limpiarMensaje();
            actualizarMatriz();
            for (int s = 0; s < 10; s++) {
            mostrar_columnas();
            delay(1);
            }
        }

        }else if (digitalRead(BTN_INI) == HIGH) {  //
        btn_pressed = true;                       //false---true
        tmp_inicio = millis();                    //10
        } else {
        btn_pressed = false;
        }
        
        if (tmp_transcurrido >= 1000 && btn_pressed == false) {
        
        limpiarMensaje();
        actualizarMatriz();
        for (int s = 0; s < 10; s++) {
            mostrar_columnas();
            delay(1);
        }
        for (int p = 0; p <= 15; p++) {
            estadoMatriz[p] = matrizPausada[p];
            matrizPausada[p] = 0;
        }  //
        inicio_pausa = 0;
        estado_juego = 1;
        }

        if(digitalRead(BTN_DER) == HIGH){
            estado_juego = 3;
            tmp_transcurrido=0;
        }

    }else if(estado_juego ==3){

        if(digitalRead(BTN_IZQ) == HIGH){
        estado_juego = 2;
        }
        activar_buzzer(5);
        delay(100);
        frecuencia_buzzer = map(analogRead(A0), 0, 1023, 0, 255);

    }


    }
```
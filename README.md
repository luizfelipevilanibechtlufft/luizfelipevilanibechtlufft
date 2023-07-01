/* @autor: Eletrogate
   @licenÃ§a: GNU GENERAL PUBLIC LICENSE Version 3 */

#include "constantes.h" // inclui cabeÃ§alho com as constantes

bool dado_novo;                                       // declara as
uint8_t v, a, r, vel_int, val_mA, val_mB, quadrante;  // variaveis que serao
char vel[4], angulo[4], recebido[16], c;              // utilizada ao longo
unsigned  ang_int;                                    // do programa

bool caractereValido(char c)  {
  return ((c >= '0' && c <= '9') || c == ' ');  // verifica se o caractere recebido do ESP Ã© um nÃºmero ou um espaÃ§o
}

void setup() {
  delay(500);                 // aguarda o sistema estabilizar
  Serial.begin(9600);         // inicia a serial
  pinMode(pinIn, INPUT);      // configura e inicia
  pinMode(pinOut, OUTPUT);    // as entradas
  digitalWrite(pinOut, LOW);  // e saÃ­das
}

void loop() {

  if(!digitalRead(pinIn)) {           // se detectar nova conexÃ£o Ã  pÃ¡gina
    digitalWrite(pinOut, LOW);        // avisa que detectou
    delay(100);                       // aguarda 100 milisegundos
    while(Serial.available())
      Serial.read();                  // esvazia o buffer
    digitalWrite(pinOut, HIGH);       // prepara para a proxima conexÃ£o
  }

  if(Serial.available() && Serial.read() == caractereInicio) {  // se hÃ¡ dados e o primeiro Ã© caractereInicio
  dado_novo = true;                                   // armazena que hÃ¡ dado novo
    r = -1;                                           // prepara o indice de recebido
    while((c = Serial.read()) != caractereFinal)      // enquanto nao for o caractere finalizador
      if(caractereValido(c))                          // se for um caractere valido
        recebido[++ r] = c;                           // o armazena a incrementa o indice

    for(v = 0; recebido[v] != caractereSepara; v ++)  // do primeiro caractere atÃ© caractereSepara
      vel[v] = recebido[v];                           // copia-o para vel
    vel[v] = '\0';                                    // insere o caractere nulo na posiÃ§Ã£o posterior Ã  do ultimo copiado

    for(a = v + 1; a <= r; a ++)                      // do primeiro apÃ³s caractereSepara atÃ© o ultimo caractere valido de recebido
      angulo[a - v - 1] = recebido[a];                // copia-o para angulo - v - 1 Ã© a diferenÃ§a entre a posiÃ§Ã£o de recebido e a de angulo

    angulo[r - v - 1 < 3 ?      // angulo recebeu atÃ© 3 caracteres?
            r - v : 3] = '\0';  // se sim, insere o caractere nulo na posiÃ§Ã£o posterior Ã  do ultimo copiado. se nÃ£o, o insere na ultima posiÃ§Ã£o de angulo
  }

  if(dado_novo) { // se recebeu um novo dado
    vel_int = atoi(vel);  vel_int = map(vel_int, 0, 100, 0, 255); // transforma a velocidade em um inteiro e, entÃ£o, o mapeia entre 0 e 255
    ang_int = atoi(angulo);                                       // transforma o angulo em um inteiro

    if(ang_int < 90) quadrante = 1; else if(ang_int < 180) quadrante = 2; else if(ang_int < 270) quadrante = 3; else if(ang_int < 360) quadrante = 4; // determina em qual quadrante o joystick estÃ¡
    
    if(quadrante == 1 || quadrante == 4) {                                      // se estiver na direita
      val_mB = (uint8_t) vel_int;                                               // o motor B recebe a velocidade indicada no joystick
      val_mA = (uint8_t) (vel_int * (1.0 - cos(ang_int * PI / 180.0) / 2.0)); } // o motor A recebe esta velocidade reduzida proporcionalmente ao cosseno do Ã¢ngulo
    else {                                                                      // caso nÃ£o
      val_mB = (uint8_t) (vel_int * (1.0 + cos(ang_int * PI / 180.0) / 2.0));   // o motor B recebe a velocidade acrescida proporcionalmente ao cosseno (que serÃ¡ negativo) do Ã¢ngulo
      val_mA = (uint8_t) vel_int; }                                             // o motor A recebe a velocidade indicada no joystick

    if(quadrante < 3) {                                 // se estiver em cima/na frente
      analogWrite(in2, val_mA); analogWrite(in1, 0);    // aciona os motores
      analogWrite(in4, val_mB); analogWrite(in3, 0);  } // para frente
    else {                                              // senÃ£o
      analogWrite(in2, 0); analogWrite(in1, val_mA);    // aciona os motores
      analogWrite(in4, 0); analogWrite(in3, val_mB);  } // para trÃ¡s
  } dado_novo = false;                                  // indica que o dado jÃ¡ foi tratado
}

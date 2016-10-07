#include <SoftwareSerial.h>
#include <SPI.h>
#include <SD.h>
//#include <math.h>

#define CS 10

SoftwareSerial gps(0,1);

File GPS_folder;
File GPS_datalog;
File GPS_config;
char directory[22];

char incomingByte;

long timer_scatto = 5000;
long n_foto = 0;

double latitude;
double longitude;
float elevation;

long current_time;
char time_file = "000000";

char date[] = "00_00_00";

bool test = false;
bool error = false;
unsigned long temp;
byte mode = 1;

/*
double toRad(double deg){
    double rad = 2 * M_PI / 360;
    return rad;
}
*/

/*
double lawOfCos(double lat1, double lat2, double lon1, double lon2){
    double d;
    double temp1;
    double temp2;

    temp1 = sin(toRad(lat1)) * sin(toRad(lat2));
    temp2 = cos(toRad(lon1)) * cos(toRad(lon2)) * cos(toRad(lon2 - lon1));

    d = acos(temp1 + temp2) * R;

    return d;
}
*/

/*
void writeConfig(){
    // Read the variable from the config.txt file.
    // Return the variable in the line given as
    // input.

    long conf_var;

    GPS_config = SD.open("config.txt", FILE_WRITE);
    Serial.println(F("> Apertura del config file per la scrittura"));
    if(!GPS_config){
        Serial.println(F("[ERROR] Config write: impossibile aprire config file"));
    }

    GPS_config.seek(0);

    Serial.println(F("Inserire il valore timer_scatto"));
    while(Serial.available() == 0){
        ;
    }
    conf_var = Serial.parseInt();

    GPS_config.find("timer=");
    GPS_config.print(conf_var);
    GPS_config.print("]   ");

    Serial.println(F("Inserire il valore n_foto"));
    while(Serial.available() == 0){
        ;
    }
    conf_var = Serial.parseInt();

    GPS_config.find("nfoto=");
    GPS_config.print(conf_var);
    GPS_config.print("]   ");

    GPS_config.close();
    Serial.println("> Salvataggio config file");
}
*/

bool validFix(){
    /************************************************
    * Return true if valid fix is available, return
    * false if no valid fix is available in a second
    *************************************************/
    if(gps.available()){
        if(gps.find(",A*")){
            return true;
        }else{
            Serial.println(F("[ERROR] Valid fix: valid fix non disponibile"));
            return false;
        }
    }else{
        Serial.println(F("[ERROR] Valid fix: gps non disponibile"));
        return false;
    }
}

bool encodeGGA(){
    /************************************************************
    * Encode GPGGA message: return true if the message was
    * correctly encoded. Save the needed data into global variable.
    * Check the Valid Fix of the message too.
    ************************************************************/
    char message [70];
    int c, i;
    int k = 0;
    int comma = 0;

    int temp_int;

    char buf10[11];
    char buf11[12];
    char buf6[7];

    if(gps.find("GGA")){
        c = gps.readBytesUntil('\r', message, 70);
    }else{
        Serial.println(F("[ERROR] GGA encoding: GGA message non trovato"));
        return false;
    }
    message[c] = '\0';
    //Serial.print("$GPGGA");
    //Serial.println(message);

    for(i = 0; (message[i] != '\0') && (comma < 10); i++){
        while(message[i] == ','){
            comma++;
            i++;
        }
        switch(comma){
            case 2: //Latitude
                if(!(((message[i] >= '0') && (message[i] <= '9')) || (message[i] == '.'))){
                    Serial.println(F("[ERROR] GGA encoding: invalid latitude"));
                    return false;
                }
                buf10[k] = message[i];
                k++;
                if(k == 10){
                    buf10[k] = '\0';
                    k = 0;
                    latitude = atof(buf10);
                    temp_int = (int)latitude / 100;
                    latitude = temp_int + (latitude - (temp_int * 100)) / 60;
                }
                break;

            case 3:
                if(message[i] == 'S'){
                    latitude = latitude * (-1);
                }
                break;

            case 4: //Longitude
                if(!(((message[i] >= '0') && (message[i] <= '9')) || (message[i] == '.'))){
                    Serial.println(F("[ERROR] GGA encoding: invalid longitude"));
                    return false;
                }

                buf11[k] = message[i];
                k++;
                if(k == 11){
                    buf11[k] = '\0';
                    k = 0;
                    longitude = atof(buf11);
                    temp_int = (int)(longitude / 100);
                    longitude = temp_int + (longitude - (temp_int * 100)) / 60;
                }
                break;

            case 5:
                if(message[i] == 'W'){
                    longitude = longitude * (-1);
                }
                break;

            case 6: //Valid Fix
                if((message[i]  != '1') && (message[i] != '2')){
                    Serial.println(F("[ERROR] GGA encoding: invalid fix"));
                    return false;
                }
                break;

            case 9: //Elevation
                if(!( ( (message[i] >= '0') && (message[i] <= '9') ) || (message[i] == '.') || (message[i] == '-'))){
                    Serial.println(F("[ERROR] GGA encoding: invalid altitude"));
                    return false;
                }
                buf6[k] = message[i];
                k++;
                if(message[i-1] == '.'){
                    buf6[k] = '\0';
                    k = 0;
                    elevation = atof(buf6);
                }
                break;
        }
    }
    return true;
}

bool encodeRMC(char *date, long *time){
    /****************************************************
    * Encode GPRMC message: return true if the message was
    * correctly encoded. Save the date in the format needed
    * for naming the working folder (gg_mm_yy).
    ****************************************************/

    int i, j, k;
    int check;
    int ciao;
    byte comma = 0;
    char message [70];
    char buf9 [10];

    if(gps.find("RMC")){
        j = gps.readBytesUntil('\r', message, 64);
    }else{
        Serial.println(F("[ERROR] RMC encoding: RMC message non trovato"));
        return false;
    }
    message[j] = '\0';
    //Serial.print("$GPRMC")
    //Serial.println(message);

    for(i = 0; (message[i] != '\0') && (comma < 10); i++){
        while(message[i] == ','){
            comma++;
            i++;
        }
        switch(comma){
            case 1: //UTC Time
                if(!(((message[i] >= '0') && (message[i] <= '9')) || (message[i] == '.'))){
                    Serial.println(F("[ERROR] RMC encoding: invalid time"));
                    return false;
                }
                buf9[k] = message[i];
                k++;
                if(k == 9){
                    buf9[k] = '\0';
                    *time = atol(buf9);
                    *time = *time + 20000;
                    if(*time >= 240000 && *time <= 245959){
                        *time = *time - 240000
                    }else if(*time >= 250000 && *time <= 255959)
                        *time = *time - 250000
                    }
                    k = 0; //Posso riutilizzarlo se necessario
                }
                break;

            case 2: //Valid Fix
                if(!(message[i] == 'A' || message[i] == 'D')){
                    Serial.println("[ERROR] RMC encoding: invalid fix");
                    return false;
                }
                break;

            case 9: //Date
                if(!((message[i] >= '0') && (message[i] <= '9'))){
                    Serial.println(F("[ERROR] RMC encoding: invalid date"));
                    return false;
                }

                if(*date == '_'){
                    date++;
                }
                *date = message[i];
                date++;
                break;
        }
    }
    return true;
}

long readConfigVar(int var){
    /*********************************************
    * Read the variable from the config.txt file.
    * Return the variable in the line given as
    * input.
    **********************************************/
    int c;
    char message[10];
    long conf_var;

    GPS_config = SD.open("config.txt", FILE_READ);
    //Serial.println("> Apertura config file per la lettura");
    if(!GPS_config){
        Serial.println(F("[ERROR] Read var: impossibile aprire config file"));
    }

    if(var == 1){
        GPS_config.find("timer=");
    }else if(var == 2){
        GPS_config.find("nfoto=");
    }

    c = GPS_config.readBytesUntil(']', message, 10);
    message[c] = '\0';

    conf_var = atol(message);

    GPS_config.close();
    return conf_var;
}

long printCoords(long numb){
    //Stampa i dati sul file di log
    GPS_datalog = SD.open(directory, FILE_WRITE);
    //Serial.println(F("> Apertura datalog per la scrittura"));
    if(!GPS_datalog){
        Serial.println(F("[ERROR] Coords print: impossibile aprire il file"));
    }

    GPS_datalog.print("IMG_"); GPS_datalog.print(numb); GPS_datalog.print(".JPG,");
    GPS_datalog.print(latitude, 4); GPS_datalog.print(",");
    GPS_datalog.print(longitude, 4); GPS_datalog.print(",");
    GPS_datalog.println(elevation, 1);

    Serial.print("IMG_"); Serial.print(numb); Serial.print(".JPG,");
    Serial.print(latitude, 4); Serial.print(",");
    Serial.print(longitude, 4); Serial.print(",");
    Serial.println(elevation, 1);

    //Chiude il file per salvarlo
    GPS_datalog.close();
    //Serial.println("> Salvataggio datalog");

    return numb+1;

}

void setup(){
    /*============================================================================*/
    // PC Serial baudrate
    Serial.begin(115200);

    pinMode(2, OUTPUT);
    pinMode(3, OUTPUT);

    while(!Serial){
        delay(500);
    }

    // GPS Serial baudrate
    gps.begin(38400);
    gps.setTimeout(1000);

    do{
        Serial.println(F("> Attendo un fix valido"));
        while(!validFix()){
            delay(500);
        }

        Serial.println(F("> Attendo data e orario validi"));
        while(!encodeRMC(date, &current_time)){
            delay(500);
        }

        Serial.println(F("> Inizializzazione SD card"));

        if(!SD.begin(CS)){
            Serial.println(F("[ERROR] Setup: inizializzazione SD card fallita"));
            digitalWrite(3, HIGH);
            error = true;
        }else{
            digitalWrite(3, LOW);
            error = false;
        }

        delay(1000);

        /*============================================================================*/
        //Lettura delle variabili da config.txt e creazione del file se non presente
        if(!error){
            if(SD.exists("config.txt")){
                Serial.println(F("> Config file gia' esistente"));
                Serial.println(F("> Estrazione valori dal file config"));
                timer_scatto = readConfigVar(1);
                //Serial.print(F("timer_scatto = "));
                //Serial.println(timer_scatto);
                n_foto = readConfigVar(2);
                //Serial.print(F("n_foto = "));
                //Serial.println(n_foto);
            }else{
                Serial.println(F("> Creazione config file"));

                GPS_config = SD.open("config.txt", FILE_WRITE);

                if(!GPS_config){
                    Serial.println(F("[ERROR] Setup: creazione config.txt fallita"));
                    digitalWrite(3, HIGH);
                    error = true;
                }else{
                    digitalWrite(3, LOW);
                    error = false;

                    GPS_config.println(F("Timer scatto automatico in millisecondi (0 con scatto manuale)"));
                    GPS_config.println(F("[timer=2000]"));
                    GPS_config.println(F("Numerazione di partenza della foto"));
                    GPS_config.println(F("[nfoto=2000]"));

                    Serial.println(F("> Salvataggio config file"));
                    GPS_config.close();
                }
            }
        }

        // Check if the current folder exists, and get the filename.
        if((!SD.exists(date)) && (!error)){
            test = SD.mkdir(date);

            Serial.print(F("> Creazione della cartella "));
            Serial.println(date);

            if(!test){
                error = true;
                digitalWrite(3, HIGH);
                Serial.println(F("[ERROR] Setup: Creazione cartella falita"));
            }else{
                error = false;
                digitalWrite(3, LOW);
            }
        }

        if(!error){
            itoa(current_time, time_file, 10);
            strcpy(directory, date);
            strcat(directory, "/");
            strcat(directory, time_file);
            strcat(directory, ".txt");

            Serial.print(F("> Creazione di "));
            Serial.print(current_time);
            Serial.println(F(".txt"));

            GPS_datalog = SD.open(directory, FILE_WRITE);

            if(!GPS_datalog){
                error = true;
                digitalWrite(3, HIGH);
                Serial.println(F("[ERROR] Setup: creazione del file di lavoro fallita"));
            }else{
                error = false;
                digitalWrite(3, LOW);
            }

            GPS_datalog.close();
        }
    }while(error == true);

    /*
    //Serial.println(F("Vuoi cambiare i valori del file di config? y/n"));
    while(Serial.available() == 0){
        ;
    }
    */


    /*
    incomingByte = Serial.read();

    if(incomingByte == 'y'){
        writeConfig();
    }
    */

    digitalWrite(2, HIGH);
    delay(2000);
    digitalWrite(2, LOW);
}

void loop(){
    /*
    if(digitalRead(9) == HIGH && mode == 0){
        if(timer_scatto != 0){
            mode = 1;
            temp = millis();
        }else{
            if(encodeGGA()){
                n_foto = printCoords(n_foto);

            }else{
                Serial.println(F("[ERROR] Main: coordinate non valide"));
            }
        }
    }
    */

    temp = millis();

    if(mode == 1){
        while(millis() - temp <= timer_scatto){}
        if(encodeGGA()){
            digitalWrite(2, HIGH);
            n_foto = printCoords(n_foto);
            digitalWrite(2, LOW);
        }else{
            Serial.println(F("[ERROR] Main: coordinate non valide"));
        }
    }

}

Berikut rangkaian Proteus menggunakan sensor Rain Sensor

![image](https://github.com/MarcellinoAditiyaYahya06/EAS_IOT_RAIN_SENSOR/assets/172895496/bde0277f-830e-4e64-a021-90256fdf7901)

Komponen yang digunakan yaitu:
•	Arduino Uno
•	ENC28J60-Ethernet
•	Logicstate
•	Rain Sensor

Berikut adalah Sourcode Arduino:

#include <EtherCard.h>

// Konfigurasi IP Statis
static byte myip[] = { 169, 254, 199, 19 }; // IP statis untuk Arduino
static byte gwip[] = { 169, 254, 199, 18 }; // IP Gateway

// Alamat MAC dari Arduino
static byte mymac[] = { 0x74, 0x69, 0x69, 0x2D, 0x30, 0x31 };

// Ukuran Buffer Ethernet
byte Ethernet::buffer[700];

void setup() {
  // Inisialisasi Ethernet dengan ukuran buffer dan alamat MAC
  ether.begin(sizeof Ethernet::buffer, mymac, SS);
  // Mengatur IP statis dan gateway
  ether.staticSetup(myip, gwip);
  // Mengatur pin 2 sebagai input untuk sensor hujan
  pinMode(2, INPUT);
}

void loop() {
  // Membaca nilai digital dari sensor hujan yang terhubung ke pin 2
  int rainValue = digitalRead(2);
  // Buffer untuk status hujan
  const char* rainStatus;

  // Menentukan status hujan berdasarkan nilai sensor
  if (rainValue == LOW) {
    rainStatus = "Hujan"; // Hujan
  } else {
    rainStatus = "Tidak Hujan"; // Tidak Hujan
  }

  // Buffer untuk respon HTML
  char htmlResponse[250];
  // Membuat respon HTML dengan status hujan
  sprintf(htmlResponse, "HTTP/1.1 200 OK\r\nContent-Type: text/html\r\nConnection: close\r\n\r\n"
                        "<html>\r\n"
                        "<head>\r\n"
                        "<meta http-equiv='refresh' content='3'>\r\n"
                        "</head>\r\n"
                        "<body>\r\n"
                        "Status Hujan: %s\r\n"
                        "</body>\r\n"
                        "</html>\r\n", rainStatus);

  // Memeriksa apakah ada paket yang masuk
  word pos = ether.packetLoop(ether.packetReceive());
  if (pos) {
    // Pointer ke data paket
    char *data = (char *)Ethernet::buffer + pos;
    // Menyalin respon HTML ke buffer Ethernet
    memcpy(ether.tcpOffset(), htmlResponse, strlen(htmlResponse));
    // Mengirim balasan HTTP server dengan panjang respon HTML
    ether.httpServerReply(strlen(htmlResponse));
  }
}


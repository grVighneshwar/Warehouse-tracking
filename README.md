# Warehouse-tracking
## Code for Aries Vega 3 board
```
// RFID Reader Pins (Manual SPI)
#define RFID1_SS 9
#define RFID1_RST 11
#define RFID2_SS 13
#define RFID2_RST 12
#define SPI_MOSI 7
#define SPI_MISO 8
#define SPI_SCK 9

int room1Count = 0;
int room2Count = 0;
String lastUID1 = "";
String lastUID2 = "";

void setup() {
  Serial.begin(115200);
  
  // Initialize SPI pins
  pinMode(SPI_MOSI, OUTPUT);
  pinMode(SPI_MISO, INPUT);
  pinMode(SPI_SCK, OUTPUT);
  
  // Initialize RFID readers
  pinMode(RFID1_SS, OUTPUT);
  pinMode(RFID2_SS, OUTPUT);
  digitalWrite(RFID1_SS, HIGH);
  digitalWrite(RFID2_SS, HIGH);
  
  // Reset RFID readers
  pinMode(RFID1_RST, OUTPUT);
  pinMode(RFID2_RST, OUTPUT);
  digitalWrite(RFID1_RST, HIGH);
  digitalWrite(RFID2_RST, HIGH);
  delay(100);
  digitalWrite(RFID1_RST, LOW);
  digitalWrite(RFID2_RST, LOW);
  delay(100);
  digitalWrite(RFID1_RST, HIGH);
  digitalWrite(RFID2_RST, HIGH);
  
  Serial.println("System Ready");
}

String readRFID(uint8_t ssPin) {
  digitalWrite(ssPin, LOW);
  String uid = "";
  
  // Manual SPI communication
  for(uint8_t i = 0; i < 10; i++) {
    uint8_t data = 0;
    for(uint8_t j = 0; j < 8; j++) {
      digitalWrite(SPI_SCK, LOW);
      digitalWrite(SPI_MOSI, (i == 0) ? 0x26 : 0x00); // RFID command
      digitalWrite(SPI_SCK, HIGH);
      data |= digitalRead(SPI_MISO) << (7 - j);
    }
    if(i < 4) uid += String(data, HEX); // Get first 4 bytes as UID
  }
  
  digitalWrite(ssPin, HIGH);
  uid.toUpperCase();
  return uid;
}

void loop() {
  // Read RFID Reader 1
  String uid1 = readRFID(RFID1_SS);
  if(uid1.length() > 0 && uid1 != lastUID1) {
    room1Count++;
    lastUID1 = uid1;
    Serial.print("Room1:");
    Serial.print(room1Count);
    Serial.print(",Room2:");
    Serial.println(room2Count);
  }

  // Read RFID Reader 2
  String uid2 = readRFID(RFID2_SS);
  if(uid2.length() > 0 && uid2 != lastUID2) {
    room2Count++;
    lastUID2 = uid2;
    Serial.print("Room1:");
    Serial.print(room1Count);
    Serial.print(",Room2:");
    Serial.println(room2Count);
  }
  
  delay(200);
}
```
## Python script which reads the data from the serial monitor and updates the gui
```
import tkinter as tk
import serial
from serial.tools import list_ports

class WarehouseApp:
    def __init__(self, master):
        self.master = master
        master.title("Warehouse Monitoring")
        
        self.room1_label = tk.Label(master, text="Room 1: 0", font=("Arial", 24))
        self.room1_label.pack(pady=20)
        
        self.room2_label = tk.Label(master, text="Room 2: 0", font=("Arial", 24))
        self.room2_label.pack(pady=20)
        
        self.connect_serial()
        
    def connect_serial(self):
        ports = list_ports.comports()
        for port in ports:
            if "USB" in port.description:
                try:
                    self.ser = serial.Serial(port.device, 115200, timeout=1)
                    self.read_serial()
                    return
                except:
                    pass
        print("No compatible device found")

    def read_serial(self):
        if self.ser.in_waiting > 0:
            data = self.ser.readline().decode('utf-8').strip()
            self.process_data(data)
        self.master.after(100, self.read_serial)

    def process_data(self, data):
        try:
            parts = data.split(',')
            room1 = parts[0].split(':')[1]
            room2 = parts[1].split(':')[1]
            self.room1_label.config(text=f"Room 1: {room1}")
            self.room2_label.config(text=f"Room 2: {room2}")
        except:
            pass

if __name__ == "__main__":
    root = tk.Tk()
    app = WarehouseApp(root)
    root.geometry("300x200")
    root.mainloop()
```
**The schematic with coustom symbols as per the design**
![image](https://github.com/user-attachments/assets/a900eae5-be26-41af-a85a-bc716adce493)
**PCB layout completely designed by me with coustom footprints as per the design**
![image](https://github.com/user-attachments/assets/c014d6b4-a76e-4977-8e52-523575c6768d)
**PCB Etched by my own hands using heat transfer method**
![WhatsApp Image 2025-05-23 at 17 35 10_5453f8b4](https://github.com/user-attachments/assets/5d500747-a57c-4f61-b21d-9bd3820e6b66)
![WhatsApp Image 2025-05-23 at 17 35 11_426a29fe](https://github.com/user-attachments/assets/841b6f11-b36c-4b13-867d-e1c17e85887d)

**working video**
https://drive.google.com/file/d/1UaoHjN2A8Xr_qGcLBaLFmA13JefEUH24/view?usp=sharing

const int STEP_PIN = 2;   
const int DIR_PIN = 3;    
const int PHOTO_PIN = A0; 

const int LIGHT_THRESHOLD = 1500; 
const int STEPS_PER_REV = 200;    
const int STEP_DELAY = 500;       

bool blindsClosed = true; 
const float GAMMA = 0.7;
const float RL10 = 50;
void setup() {
  pinMode(STEP_PIN, OUTPUT);
  pinMode(DIR_PIN, OUTPUT);
  Serial.begin(9600);
}

void stepMotor(int steps, bool direction) {
  digitalWrite(DIR_PIN, direction);
  for (int i = 0; i < abs(steps); i++) {
    digitalWrite(STEP_PIN, HIGH);
    delayMicroseconds(STEP_DELAY);
    digitalWrite(STEP_PIN, LOW);
    delayMicroseconds(STEP_DELAY);
  }
}

void closeBlinds() {
  if (!blindsClosed) {
    stepMotor(STEPS_PER_REV, HIGH); 
    blindsClosed = true;
    Serial.println("Blinds closed.");
  }
}

void openBlinds() {
  if (blindsClosed) {
    stepMotor(STEPS_PER_REV, LOW); 
    blindsClosed = false;
    Serial.println("Blinds opened.");
  }
}

void loop() {

  int analogValue = analogRead(PHOTO_PIN);
  float voltage = analogValue / 1024. * 5;
  float resistance = 2000 * voltage / (1 - voltage / 5);
  float lightLevel = pow(RL10 * 1e3 * pow(10, GAMMA) / resistance, (1 / GAMMA));
  //int lightLevel = analogRead(PHOTO_PIN);
  Serial.print("Light level: ");
  Serial.println(lightLevel);

  if (lightLevel > LIGHT_THRESHOLD) 
  {
    if(blindsClosed == true)
    {
      stepMotor(200, HIGH); 
      blindsClosed = false;
    }
  } 
  if (lightLevel <= LIGHT_THRESHOLD) 
  {
   if(blindsClosed == false)
    {
      stepMotor(200, LOW); 
      blindsClosed = true;
    }
  }

  delay(100); 
}

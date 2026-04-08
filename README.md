# Keyword-Spotting-KWS-on-Embedded-Systems
/*
 * Keyword Spotting (KWS) - Final Assignment #2
 * Targets: Clap, Snap, and Tap sounds.
 * Hardware: Arduino Nano 33 BLE Sense.
 */

#include <PDM.h>
#include <TensorFlowLite.h>
#include <tensorflow/lite/micro/all_ops_resolver.h>
#include <tensorflow/lite/micro/micro_interpreter.h>
#include <tensorflow/lite/schema/schema_generated.h>
#include "model.h" // Your exported model from Colab

// Buffer to read samples into, each sample is 16-bits (PCM)
short sampleBuffer[256]; [cite: 221]
volatile int samplesRead; [cite: 223]

// TFLite Global Variables
tflite::AllOpsResolver tflOpsResolver;
const tflite::Model* tflModel = nullptr;
tflite::MicroInterpreter* tflInterpreter = nullptr;
TfLiteTensor* tflInputTensor = nullptr;
TfLiteTensor* tflOutputTensor = nullptr;

// Allocate memory for TFLite Micro (Arena Size)
constexpr int tensorArenaSize = 64 * 1024; // Adjust based on your model size
byte tensorArena[tensorArenaSize] __attribute__((aligned(16)));

// Labels for your classes [cite: 2221, 2222, 2223]
const char* KEYWORDS[] = {
  "Clap",
  "Snap",
  "Tap"
};
#define NUM_KEYWORDS (sizeof(KEYWORDS) / sizeof(KEYWORDS[0]))

void setup() {
  Serial.begin(9600);
  while (!Serial);

  // 1. Initialize Microphone (PDM) at 16 kHz mono [cite: 230, 231]
  PDM.onReceive(onPDMdata); [cite: 225]
  if (!PDM.begin(1, 16000)) { [cite: 231]
    Serial.println("Failed to start PDM!");
    while (1);
  }

  // 2. Load TFLite Model [cite: 1881]
  tflModel = tflite::GetModel(model);
  if (tflModel->version() != 3) {
    Serial.println("Model schema mismatch!");
    while (1);
  }

  // 3. Setup Interpreter (Using nullptr for compatibility with KAIST/Harvard libraries)
  static tflite::MicroInterpreter static_interpreter(
      tflModel, tflOpsResolver, tensorArena, tensorArenaSize, nullptr, nullptr);
  tflInterpreter = &static_interpreter;

  // 4. Allocate memory for model's tensors
  tflInterpreter->AllocateTensors();
  tflInputTensor = tflInterpreter->input(0);
  tflOutputTensor = tflInterpreter->output(0);
}

void loop() {
  // Wait for microphone data
  if (samplesRead) {
    // In a full KWS pipeline, you would perform MFCC extraction here [cite: 712, 713]
    // For this GitHub version, we assume the input is ready after 1 second of audio [cite: 2215]
    
    // Fill Input Tensor (Simplified)
    for (int i = 0; i < tflInputTensor->bytes / sizeof(float); i++) {
        tflInputTensor->data.f[i] = (float)sampleBuffer[i] / 32768.0; // Normalizing PCM [cite: 186]
    }

    // 5. Run Inference [cite: 2107]
    TfLiteStatus invokeStatus = tflInterpreter->Invoke();
    if (invokeStatus != kTfLiteOk) {
      Serial.println("Inference failed!");
      return;
    }

    // 6. Print Results [cite: 2108]
    Serial.println("--- Prediction Results ---");
    for (int i = 0; i < NUM_KEYWORDS; i++) {
      Serial.print(KEYWORDS[i]);
      Serial.print(": ");
      Serial.println(tflOutputTensor->data.f[i], 4); // Probability score
    }
    
    samplesRead = 0; // Reset for next sound window
  }
}

// Callback function to read data from PDM microphone [cite: 236]
void onPDMdata() {
  int bytesAvailable = PDM.available(); [cite: 238]
  int bytesRead = PDM.read(sampleBuffer, bytesAvailable); [cite: 240]
  samplesRead = bytesRead / 2; // 16-bit PCM (2 bytes per sample) [cite: 241, 242]
}

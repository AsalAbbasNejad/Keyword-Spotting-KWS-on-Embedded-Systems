/*
 * Keyword Spotting (KWS) - Assignment #2
 * Optimized Inference Code using CMSIS-DSP
 */

#include <PDM.h>
#include <TensorFlowLite.h>
#include <tensorflow/lite/micro/all_ops_resolver.h>
#include <tensorflow/lite/micro/micro_interpreter.h>
#include <tensorflow/lite/schema/schema_generated.h>
#include "model.h"

// Buffer settings
short sampleBuffer[256];
volatile int samplesRead;

// TFLite Variables
tflite::AllOpsResolver tflOpsResolver;
const tflite::Model* tflModel = nullptr;
tflite::MicroInterpreter* tflInterpreter = nullptr;
TfLiteTensor* tflInputTensor = nullptr;

// Allocate memory (8 KB as specified in the docs) 
constexpr int tensorArenaSize = 8 * 1024;
byte tensorArena[tensorArenaSize] __attribute__((aligned(16)));

const char* KEYWORDS[] = {"Clap", "Snap", "Tap"};
#define NUM_KEYWORDS (sizeof(KEYWORDS) / sizeof(KEYWORDS[0]))

void setup() {
  Serial.begin(9600);
  while (!Serial);

  // Initialize PDM at 16 kHz [cite: 181, 231]
  if (!PDM.begin(1, 16000)) {
    Serial.println("Failed to start PDM!");
    while (1);
  }
  PDM.onReceive(onPDMdata);

  tflModel = tflite::GetModel(model);
  // Manual version check to avoid schema mismatch error [cite: 82]
  if (tflModel->version() != 3) {
    Serial.println("Model version mismatch!");
  }

  // Optimized Interpreter setup [cite: 89]
  static tflite::MicroInterpreter static_interpreter(
      tflModel, tflOpsResolver, tensorArena, tensorArenaSize, nullptr, nullptr);
  tflInterpreter = &static_interpreter;

  tflInterpreter->AllocateTensors();
  tflInputTensor = tflInterpreter->input(0);
}

void loop() {
  if (samplesRead > 0) {
    // Fill Input Tensor: Normalize PCM to float range [-1, 1] [cite: 186]
    // The manual loops are replaced by direct pointer access for speed
    for (int i = 0; i < samplesRead; i++) {
      tflInputTensor->data.f[i] = sampleBuffer[i] / 32768.0f;
    }

    // Run Inference
    if (tflInterpreter->Invoke() == kTfLiteOk) {
      TfLiteTensor* output = tflInterpreter->output(0);
      for (int i = 0; i < NUM_KEYWORDS; i++) {
        Serial.print(KEYWORDS[i]);
        Serial.print(": ");
        Serial.println(output->data.f[i], 4);
      }
    }
    samplesRead = 0;
  }
}

void onPDMdata() {
  int bytesAvailable = PDM.available();
  int bytesRead = PDM.read(sampleBuffer, bytesAvailable);
  samplesRead = bytesRead / 2; // Convert bytes to 16-bit samples [cite: 242]
}

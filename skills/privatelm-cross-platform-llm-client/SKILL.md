---
name: privatelm-cross-platform-llm-client
description: A unified Flutter-based AI client supporting local on-device GGUF model inference and cloud API fallback for building privacy-focused LLM applications.
triggers:
  - how do I integrate PrivateLM into my Flutter app
  - set up local LLM inference on Android with PrivateLM
  - configure cloud API fallback in PrivateLM
  - run GGUF models on device with Flutter
  - implement multimodal chat with local and cloud models
  - debug local inference issues in PrivateLM
  - optimize PrivateLM for low-end Android devices
  - switch between local and cloud LLM providers
---

# PrivateLM Cross-Platform LLM Client

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## What It Does

PrivateLM is a production-ready Flutter framework for building AI chat applications that seamlessly switch between:

- **Local on-device inference** — GPU-accelerated GGUF model execution via `llama.cpp` (Android/iOS)
- **Cloud API providers** — OpenAI, Anthropic Claude, Google Gemini, Kimi (Moonshot AI)
- **Multimodal capabilities** — Text and vision support for both local (Qwen2-VL) and cloud models
- **Offline-first architecture** — All data persisted locally via Hive; no cloud dependency for local mode

**Key Features:**
- Auto-detects device RAM/GPU and configures optimal inference parameters
- Background service integration with Firebase Cloud Messaging
- Cross-platform (Android full support, iOS via Metal, Web cloud-only)
- Persistent chat sessions and task management

## Installation

### Add to Flutter Project

```yaml
# pubspec.yaml
dependencies:
  flutter:
    sdk: flutter
  get: ^4.6.5
  hive: ^2.2.3
  hive_flutter: ^1.1.0
  dio: ^5.3.2
  http: ^1.1.0
  flutter_background_service: ^5.0.0
  flutter_local_notifications: ^15.1.0
  firebase_core: ^2.15.0
  firebase_messaging: ^14.6.5
  device_info_plus: ^9.0.3
  path_provider: ^2.1.0
  image_picker: ^1.0.2
  permission_handler: ^11.0.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  hive_generator: ^2.0.0
  build_runner: ^2.4.6
```

### Platform Configuration

**Android (android/app/build.gradle.kts):**
```kotlin
android {
    namespace = "com.yourcompany.yourapp"
    compileSdk = 34
    ndkVersion = "25.1.8937393"

    defaultConfig {
        minSdk = 28  // Required for llama_flutter_android
        targetSdk = 34
        ndk {
            abiFilters.add("arm64-v8a")  // 64-bit ARM only
        }
    }

    buildTypes {
        release {
            isMinifyEnabled = true
            isShrinkResources = true
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
    }
}
```

**iOS (ios/Podfile):**
```ruby
platform :ios, '12.0'

target 'Runner' do
  use_frameworks!
  use_modular_headers!
  
  # Ensure Metal framework for GPU acceleration
  pod 'MetalKit'
end
```

### Initialize the App

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:get/get.dart';
import 'package:hive_flutter/hive_flutter.dart';
import 'package:firebase_core/firebase_core.dart';
import 'services/hive_service.dart';
import 'services/device_info_service.dart';
import 'controllers/settings_controller.dart';
import 'controllers/model_controller.dart';
import 'controllers/chat_controller.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  // Initialize Firebase
  await Firebase.initializeApp();
  
  // Initialize Hive
  await Hive.initFlutter();
  await HiveService.init();
  
  // Initialize GetX controllers
  Get.put(DeviceInfoService());
  Get.put(SettingsController());
  Get.put(ModelController());
  Get.put(ChatController());
  
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return GetMaterialApp(
      title: 'PrivateLM Chat',
      theme: ThemeData.dark(),
      home: HomeView(),
    );
  }
}
```

## Architecture Overview

### Service Layer

```dart
// lib/services/inference_service.dart
import 'package:get/get.dart';
import 'dart:io' show Platform;

class InferenceService extends GetxService {
  static bool get supportsLocalInference {
    if (Platform.isAndroid) return true;
    if (Platform.isIOS) return true;
    return false;  // Web not yet supported
  }

  Future<void> loadModel(String modelPath, {
    required int contextSize,
    required int numThreads,
    required int gpuLayers,
  }) async {
    if (!supportsLocalInference) {
      throw UnsupportedError('Local inference not available on this platform');
    }
    
    // Platform-specific implementation
    if (Platform.isAndroid || Platform.isIOS) {
      await _loadModelNative(modelPath, contextSize, numThreads, gpuLayers);
    }
  }

  Stream<String> generateChat({
    required List<Map<String, String>> messages,
    required String template,
    int maxTokens = 512,
    double temperature = 0.7,
  }) async* {
    // Streaming token generation
    // Implementation calls native llama.cpp bridge
  }
}
```

### Cloud Service Integration

```dart
// lib/services/cloud_service.dart
import 'package:dio/dio.dart';
import 'package:get/get.dart';

class CloudService extends GetxService {
  final Dio _dio = Dio();

  // OpenAI-compatible endpoint
  Stream<String> generateOpenAI({
    required String apiKey,
    required List<Map<String, dynamic>> messages,
    String model = 'gpt-4',
    int maxTokens = 1024,
    double temperature = 0.7,
  }) async* {
    final response = await _dio.post(
      'https://api.openai.com/v1/chat/completions',
      options: Options(
        headers: {
          'Authorization': 'Bearer $apiKey',
          'Content-Type': 'application/json',
        },
      ),
      data: {
        'model': model,
        'messages': messages,
        'max_tokens': maxTokens,
        'temperature': temperature,
        'stream': true,
      },
    );
    
    // Parse SSE stream
    await for (final chunk in response.data.stream) {
      yield _parseOpenAIChunk(chunk);
    }
  }

  // Anthropic Claude endpoint
  Stream<String> generateClaude({
    required String apiKey,
    required List<Map<String, dynamic>> messages,
    String model = 'claude-3-opus-20240229',
    int maxTokens = 1024,
  }) async* {
    // Extract system message
    String? systemMessage;
    final userMessages = messages.where((m) {
      if (m['role'] == 'system') {
        systemMessage = m['content'];
        return false;
      }
      return true;
    }).toList();

    final response = await _dio.post(
      'https://api.anthropic.com/v1/messages',
      options: Options(
        headers: {
          'x-api-key': apiKey,
          'anthropic-version': '2023-06-01',
          'Content-Type': 'application/json',
        },
      ),
      data: {
        'model': model,
        'max_tokens': maxTokens,
        'messages': userMessages,
        if (systemMessage != null) 'system': systemMessage,
        'stream': true,
      },
    );
    
    await for (final chunk in response.data.stream) {
      yield _parseClaudeChunk(chunk);
    }
  }

  // Google Gemini endpoint
  Stream<String> generateGemini({
    required String apiKey,
    required List<Map<String, dynamic>> messages,
    String model = 'gemini-1.5-pro',
  }) async* {
    final contents = messages.map((m) => {
      'role': m['role'] == 'assistant' ? 'model' : 'user',
      'parts': [{'text': m['content']}],
    }).toList();

    final response = await _dio.post(
      'https://generativelanguage.googleapis.com/v1beta/models/$model:streamGenerateContent?key=$apiKey',
      options: Options(
        headers: {'Content-Type': 'application/json'},
      ),
      data: {'contents': contents},
    );
    
    await for (final chunk in response.data.stream) {
      yield _parseGeminiChunk(chunk);
    }
  }
}
```

## Device Detection & Auto-Configuration

```dart
// lib/services/device_info_service.dart
import 'package:device_info_plus/device_info_plus.dart';
import 'package:get/get.dart';
import 'dart:io';

enum DeviceTier { ultra, high, mid, low }

class DeviceInfoService extends GetxService {
  final DeviceInfoPlugin _deviceInfo = DeviceInfoPlugin();
  
  DeviceTier? _tier;
  int? _totalRamMB;
  
  DeviceTier get tier => _tier ?? DeviceTier.mid;
  int get totalRamMB => _totalRamMB ?? 4096;
  
  Future<void> detectDevice() async {
    if (Platform.isAndroid) {
      final androidInfo = await _deviceInfo.androidInfo;
      _totalRamMB = _estimateAndroidRAM(androidInfo);
    } else if (Platform.isIOS) {
      final iosInfo = await _deviceInfo.iosInfo;
      _totalRamMB = _estimateIOSRAM(iosInfo);
    }
    
    _tier = _calculateTier(_totalRamMB!);
  }
  
  DeviceTier _calculateTier(int ramMB) {
    if (ramMB >= 12000) return DeviceTier.ultra;  // 12GB+
    if (ramMB >= 8000) return DeviceTier.high;    // 8-12GB
    if (ramMB >= 6000) return DeviceTier.mid;     // 6-8GB
    return DeviceTier.low;                         // <6GB
  }
  
  Map<String, int> getOptimalSettings() {
    switch (tier) {
      case DeviceTier.ultra:
        return {
          'contextSize': 8192,
          'threads': 8,
          'gpuLayers': 33,
          'batchSize': 512,
        };
      case DeviceTier.high:
        return {
          'contextSize': 4096,
          'threads': 6,
          'gpuLayers': 25,
          'batchSize': 256,
        };
      case DeviceTier.mid:
        return {
          'contextSize': 2048,
          'threads': 4,
          'gpuLayers': 15,
          'batchSize': 128,
        };
      case DeviceTier.low:
        return {
          'contextSize': 1024,
          'threads': 2,
          'gpuLayers': 8,
          'batchSize': 64,
        };
    }
  }
}
```

## Local Model Management

```dart
// lib/controllers/model_controller.dart
import 'package:get/get.dart';
import 'package:dio/dio.dart';
import 'package:path_provider/path_provider.dart';
import 'dart:io';

class ModelController extends GetxController {
  final RxList<LocalModel> models = <LocalModel>[].obs;
  final RxMap<String, double> downloadProgress = <String, double>{}.obs;
  
  Future<void> downloadModel({
    required String url,
    required String modelName,
    required String fileName,
  }) async {
    final appDir = await getApplicationDocumentsDirectory();
    final modelDir = Directory('${appDir.path}/models');
    if (!await modelDir.exists()) {
      await modelDir.create(recursive: true);
    }
    
    final filePath = '${modelDir.path}/$fileName';
    
    try {
      await Dio().download(
        url,
        filePath,
        onReceiveProgress: (received, total) {
          if (total != -1) {
            final progress = received / total;
            downloadProgress[modelName] = progress;
          }
        },
      );
      
      // Save model metadata
      final model = LocalModel(
        name: modelName,
        path: filePath,
        sizeBytes: File(filePath).lengthSync(),
        downloadDate: DateTime.now(),
      );
      
      models.add(model);
      await _saveModelsToHive();
      
      Get.snackbar('Success', '$modelName downloaded successfully');
    } catch (e) {
      Get.snackbar('Error', 'Failed to download $modelName: $e');
      downloadProgress.remove(modelName);
    }
  }
  
  Future<void> deleteModel(String modelName) async {
    final model = models.firstWhere((m) => m.name == modelName);
    final file = File(model.path);
    
    if (await file.exists()) {
      await file.delete();
    }
    
    models.removeWhere((m) => m.name == modelName);
    await _saveModelsToHive();
  }
}

class LocalModel {
  final String name;
  final String path;
  final int sizeBytes;
  final DateTime downloadDate;
  
  LocalModel({
    required this.name,
    required this.path,
    required this.sizeBytes,
    required this.downloadDate,
  });
}
```

## Chat Implementation

```dart
// lib/controllers/chat_controller.dart
import 'package:get/get.dart';
import '../services/inference_service.dart';
import '../services/cloud_service.dart';
import '../services/hive_service.dart';

class ChatController extends GetxController {
  final InferenceService _inference = Get.find();
  final CloudService _cloud = Get.find();
  final HiveService _hive = Get.find();
  
  final RxList<ChatMessage> messages = <ChatMessage>[].obs;
  final RxBool isGenerating = false.obs;
  final RxString currentProvider = 'local'.obs;  // 'local', 'openai', 'claude', etc.
  
  Future<void> sendMessage(String text, {String? imagePath}) async {
    // Add user message
    final userMsg = ChatMessage(
      role: 'user',
      content: text,
      timestamp: DateTime.now(),
      imagePath: imagePath,
    );
    messages.add(userMsg);
    
    // Create assistant message placeholder
    final assistantMsg = ChatMessage(
      role: 'assistant',
      content: '',
      timestamp: DateTime.now(),
    );
    messages.add(assistantMsg);
    
    isGenerating.value = true;
    
    try {
      if (currentProvider.value == 'local') {
        await _generateLocal(assistantMsg);
      } else {
        await _generateCloud(assistantMsg);
      }
    } catch (e) {
      assistantMsg.content = 'Error: $e';
    } finally {
      isGenerating.value = false;
      await _saveConversation();
    }
  }
  
  Future<void> _generateLocal(ChatMessage assistantMsg) async {
    final chatHistory = messages
        .where((m) => m.role != 'system')
        .map((m) => {
          'role': m.role,
          'content': m.content,
        })
        .toList();
    
    await for (final token in _inference.generateChat(
      messages: chatHistory,
      template: 'chatml',  // or 'llama3', 'gemma', 'phi'
      maxTokens: 512,
      temperature: 0.7,
    )) {
      assistantMsg.content += token;
      messages.refresh();  // Trigger UI update
    }
  }
  
  Future<void> _generateCloud(ChatMessage assistantMsg) async {
    final apiKey = _hive.getApiKey(currentProvider.value);
    if (apiKey == null) {
      throw Exception('API key not configured for ${currentProvider.value}');
    }
    
    final chatHistory = messages
        .where((m) => m != assistantMsg)
        .map((m) => {
          'role': m.role,
          'content': m.content,
        })
        .toList();
    
    Stream<String> stream;
    
    switch (currentProvider.value) {
      case 'openai':
        stream = _cloud.generateOpenAI(
          apiKey: apiKey,
          messages: chatHistory,
          model: 'gpt-4',
        );
        break;
      case 'claude':
        stream = _cloud.generateClaude(
          apiKey: apiKey,
          messages: chatHistory,
          model: 'claude-3-opus-20240229',
        );
        break;
      case 'gemini':
        stream = _cloud.generateGemini(
          apiKey: apiKey,
          messages: chatHistory,
        );
        break;
      default:
        throw Exception('Unknown provider: ${currentProvider.value}');
    }
    
    await for (final token in stream) {
      assistantMsg.content += token;
      messages.refresh();
    }
  }
  
  Future<void> switchProvider(String provider) async {
    currentProvider.value = provider;
    
    // If switching to local, ensure model is loaded
    if (provider == 'local') {
      final modelController = Get.find<ModelController>();
      if (modelController.models.isEmpty) {
        Get.snackbar('Warning', 'No local models available. Download a model first.');
      }
    }
  }
}

class ChatMessage {
  final String role;
  String content;
  final DateTime timestamp;
  final String? imagePath;
  
  ChatMessage({
    required this.role,
    required this.content,
    required this.timestamp,
    this.imagePath,
  });
}
```

## Multimodal (Vision) Support

```dart
// lib/services/vision_service.dart
import 'dart:convert';
import 'dart:io';
import 'package:get/get.dart';

class VisionService extends GetxService {
  // For local models (Qwen2-VL)
  Future<String> processImageLocal(String imagePath, String prompt) async {
    final bytes = await File(imagePath).readAsBytes();
    final base64Image = base64Encode(bytes);
    
    final inference = Get.find<InferenceService>();
    
    final messages = [
      {
        'role': 'user',
        'content': [
          {'type': 'image', 'data': base64Image},
          {'type': 'text', 'text': prompt},
        ],
      },
    ];
    
    String response = '';
    await for (final token in inference.generateChat(
      messages: messages,
      template: 'qwen2vl',
    )) {
      response += token;
    }
    
    return response;
  }
  
  // For cloud providers (OpenAI GPT-4V, Gemini)
  Future<String> processImageCloud({
    required String provider,
    required String apiKey,
    required String imagePath,
    required String prompt,
  }) async {
    final bytes = await File(imagePath).readAsBytes();
    final base64Image = base64Encode(bytes);
    
    final cloud = Get.find<CloudService>();
    
    if (provider == 'openai') {
      final messages = [
        {
          'role': 'user',
          'content': [
            {'type': 'text', 'text': prompt},
            {
              'type': 'image_url',
              'image_url': {
                'url': 'data:image/jpeg;base64,$base64Image',
              },
            },
          ],
        },
      ];
      
      String response = '';
      await for (final token in cloud.generateOpenAI(
        apiKey: apiKey,
        messages: messages,
        model: 'gpt-4-vision-preview',
      )) {
        response += token;
      }
      return response;
    } else if (provider == 'gemini') {
      // Gemini handles images inline
      final messages = [
        {
          'role': 'user',
          'content': prompt,
          'image': base64Image,
        },
      ];
      
      String response = '';
      await for (final token in cloud.generateGemini(
        apiKey: apiKey,
        messages: messages,
      )) {
        response += token;
      }
      return response;
    }
    
    throw Exception('Vision not supported for provider: $provider');
  }
}
```

## Settings & Configuration

```dart
// lib/controllers/settings_controller.dart
import 'package:get/get.dart';
import '../services/hive_service.dart';
import '../services/device_info_service.dart';

class SettingsController extends GetxController {
  final HiveService _hive = Get.find();
  final DeviceInfoService _deviceInfo = Get.find();
  
  // Inference settings
  final RxInt contextSize = 2048.obs;
  final RxInt numThreads = 4.obs;
  final RxInt gpuLayers = 15.obs;
  final RxInt maxTokens = 512.obs;
  final RxDouble temperature = 0.7.obs;
  
  // API keys
  final RxMap<String, String> apiKeys = <String, String>{}.obs;
  
  @override
  void onInit() {
    super.onInit();
    loadSettings();
  }
  
  Future<void> loadSettings() async {
    // Load from Hive or use auto-detected defaults
    final savedSettings = _hive.getSettings();
    
    if (savedSettings != null) {
      contextSize.value = savedSettings['contextSize'];
      numThreads.value = savedSettings['numThreads'];
      gpuLayers.value = savedSettings['gpuLayers'];
      maxTokens.value = savedSettings['maxTokens'];
      temperature.value = savedSettings['temperature'];
    } else {
      // Auto-configure based on device
      await _deviceInfo.detectDevice();
      final optimal = _deviceInfo.getOptimalSettings();
      
      contextSize.value = optimal['contextSize']!;
      numThreads.value = optimal['threads']!;
      gpuLayers.value = optimal['gpuLayers']!;
      maxTokens.value = 512;
      temperature.value = 0.7;
      
      await saveSettings();
    }
    
    // Load API keys
    apiKeys.value = _hive.getAllApiKeys();
  }
  
  Future<void> saveSettings() async {
    await _hive.saveSettings({
      'contextSize': contextSize.value,
      'numThreads': numThreads.value,
      'gpuLayers': gpuLayers.value,
      'maxTokens': maxTokens.value,
      'temperature': temperature.value,
    });
  }
  
  Future<void> setApiKey(String provider, String key) async {
    apiKeys[provider] = key;
    await _hive.saveApiKey(provider, key);
  }
  
  void resetToAutoDetected() {
    final optimal = _deviceInfo.getOptimalSettings();
    contextSize.value = optimal['contextSize']!;
    numThreads.value = optimal['threads']!;
    gpuLayers.value = optimal['gpuLayers']!;
    saveSettings();
  }
}
```

## Configuration File Example

```dart
// lib/config/app_config.dart
class AppConfig {
  // Cloud API endpoints
  static const String openaiEndpoint = 'https://api.openai.com/v1';
  static const String anthropicEndpoint = 'https://api.anthropic.com/v1';
  static const String geminiEndpoint = 'https://generativelanguage.googleapis.com/v1beta';
  static const String kimiEndpoint = 'https://api.moonshot.cn/v1';
  
  // Model recommendations by device tier
  static const Map<String, List<String>> recommendedModels = {
    'ultra': [
      'Qwen2.5-14B-Q5_K_M.gguf',
      'Llama-3.2-11B-Vision-Q5_K_M.gguf',
      'Mistral-7B-Instruct-Q8_0.gguf',
    ],
    'high': [
      'Qwen2.5-7B-Q5_K_M.gguf',
      'Phi-3.5-mini-instruct-Q6_K.gguf',
      'Gemma-2-9B-Q4_K_M.gguf',
    ],
    'mid': [
      'Qwen2-3B-Q5_K_M.gguf',
      'Phi-3-mini-Q4_K_M.gguf',
      'TinyLlama-1.1B-Q8_0.gguf',
    ],
    'low': [
      'Qwen2-1.5B-Q4_K_M.gguf',
      'TinyLlama-1.1B-Q4_K_M.gguf',
    ],
  };
  
  // Chat templates
  static const Map<String, String> chatTemplates = {
    'chatml': '<|im_start|>system\n{system}<|im_end|>\n<|im_start|>user\n{user}<|im_end|>\n<|im_start|>assistant\n',
    'llama3': '<|begin_of_text|><|start_header_id|>system<|end_header_id|>\n{system}<|eot_id|><|start_header_id|>user<|end_header_id|>\n{user}<|eot_id|><|start_header_id|>assistant<|end_header_id|>',
    'gemma': '<bos><start_of_turn>user\n{user}<end_of_turn>\n<start_of_turn>model\n',
    'phi': '<|system|>\n{system}<|end|>\n<|user|>\n{user}<|end|>\n<|assistant|>',
  };
  
  // Environment variable keys
  static const String envOpenAIKey = 'OPENAI_API_KEY';
  static const String envAnthropicKey = 'ANTHROPIC_API_KEY';
  static const String envGeminiKey = 'GEMINI_API_KEY';
  static const String envKimiKey = 'KIMI_API_KEY';
}
```

## Common Patterns

### Pattern 1: Fallback from Local to Cloud

```dart
Future<String> generateWithFallback(String prompt) async {
  final chatController = Get.find<ChatController>();
  
  try {
    // Try local first
    chatController.switchProvider('local');
    await chatController.sendMessage(prompt);
    return chatController.messages.last.content;
  } catch (localError) {
    print('Local inference failed: $localError');
    
    // Fallback to cloud
    try {
      chatController.switchProvider('openai');
      await chatController.sendMessage(prompt);
      return chatController.messages.last.content;
    } catch (cloudError) {
      throw Exception('Both local and cloud inference failed');
    }
  }
}
```

### Pattern 2: Batch Processing with Progress

```dart
Future<void> processBatchQuestions(List<String> questions) async {
  final chatController = Get.find<ChatController>();
  final results = <String>[];
  
  for (int i = 0; i < questions.length; i++) {
    await chatController.sendMessage(questions[i]);
    results.add(chatController.messages.last.content);
    
    // Update progress
    print('Processed ${i + 1}/${questions.length}');
    
    // Rate limiting for cloud APIs
    if (chatController.currentProvider.value != 'local') {
      await Future.delayed(Duration(seconds: 1));
    }
  }
  
  return results;
}
```

### Pattern 3: Custom Model Loading with Validation

```dart
Future<void> loadAndValidateModel(String modelPath) async {
  final inference = Get.find<InferenceService>();
  final settings = Get.find<SettingsController>();
  
  // Check file exists
  final file = File(modelPath);
  if (!await file.exists()) {
    throw Exception('Model file not found: $modelPath');
  }
  
  // Check file size vs available RAM
  final fileSizeMB = (await file.length()) / (1024 * 1024);
  final deviceInfo = Get.find<DeviceInfoService>();
  final availableRAM = deviceInfo.totalRamMB * 0.7;  // Use 70% max
  
  if (fileSizeMB > availableRAM) {
    throw Exception('Model too large for device. Need ${fileSizeMB}MB, have ${availableRAM}MB');
  }
  
  // Load with optimal settings
  await inference.loadModel(
    modelPath,
    contextSize: settings.contextSize.value,
    numThreads: settings.numThreads.value,
    gpuLayers: settings.gpuLayers.value,
  );
  
  // Test generation
  final testResult = await inference.generateChat(
    messages: [{'role': 'user', 'content': 'Hi'}],
    template: 'chatml',
    maxTokens: 10,
  ).first;
  
  if (testResult.isEmpty) {
    throw Exception('Model loaded but failed to generate');
  }
  
  print('Model validated successfully');
}
```

### Pattern 4: Persistent Chat Sessions

```dart
// Save and restore conversations
class ChatSessionManager {
  final HiveService _hive = Get.find();
  
  Future<void> saveSession(String sessionId, List<ChatMessage> messages) async {
    final data = messages.map((m) => {
      

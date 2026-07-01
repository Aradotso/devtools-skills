---
name: clipboard-sync-cross-device
description: Sync clipboard content across multiple devices (PC, Android, iOS) via cloud with encryption, history, and push notifications
triggers:
  - sync clipboard across devices
  - set up clipboard sharing between computer and phone
  - configure cross-device clipboard sync
  - implement clipboard history with cloud storage
  - sync text and files via clipboard
  - enable encrypted clipboard sync
  - build clipboard synchronization tool
  - create multi-device clipboard app
---

# Clipboard Sync Cross-Device Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

## Overview

Clipboard Sync is a Python-based tool for synchronizing clipboard content across multiple devices (Windows, Android, iOS) using cloud storage (Google Drive/Dropbox). It provides end-to-end encryption, clipboard history, push notifications, and selective sync capabilities.

## Installation

```bash
# Clone the repository
git clone https://github.com/mariusboiti/Clipboard_Sync.git
cd Clipboard_Sync

# Install dependencies
pip install -r requirements.txt

# Or install specific dependencies
pip install pyperclip watchdog pycryptodome google-auth google-auth-oauthlib google-auth-httplib2 google-api-python-client dropbox pyfcm pillow
```

## Core Components

### Basic Clipboard Monitor

```python
import pyperclip
import time
import hashlib

class ClipboardMonitor:
    def __init__(self, callback=None):
        self.last_content = ""
        self.callback = callback
    
    def start_monitoring(self):
        """Monitor clipboard for changes"""
        while True:
            try:
                current_content = pyperclip.paste()
                if current_content != self.last_content:
                    self.last_content = current_content
                    if self.callback:
                        self.callback(current_content)
            except Exception as e:
                print(f"Error monitoring clipboard: {e}")
            time.sleep(1)

# Usage
def on_clipboard_change(content):
    print(f"Clipboard changed: {content[:50]}...")

monitor = ClipboardMonitor(callback=on_clipboard_change)
monitor.start_monitoring()
```

### Encryption Module

```python
from Crypto.Cipher import AES
from Crypto.Random import get_random_bytes
from Crypto.Protocol.KDF import PBKDF2
import base64
import os

class ClipboardEncryption:
    def __init__(self, password=None):
        self.password = password or os.getenv('CLIPBOARD_SYNC_PASSWORD', 'default_password')
        self.salt = get_random_bytes(16)
    
    def _get_key(self, salt):
        """Derive encryption key from password"""
        return PBKDF2(self.password, salt, dkLen=32)
    
    def encrypt(self, data):
        """Encrypt clipboard data"""
        if isinstance(data, str):
            data = data.encode('utf-8')
        
        salt = get_random_bytes(16)
        key = self._get_key(salt)
        cipher = AES.new(key, AES.MODE_GCM)
        ciphertext, tag = cipher.encrypt_and_digest(data)
        
        # Combine salt, nonce, tag, and ciphertext
        encrypted_data = salt + cipher.nonce + tag + ciphertext
        return base64.b64encode(encrypted_data).decode('utf-8')
    
    def decrypt(self, encrypted_data):
        """Decrypt clipboard data"""
        encrypted_data = base64.b64decode(encrypted_data)
        
        # Extract components
        salt = encrypted_data[:16]
        nonce = encrypted_data[16:32]
        tag = encrypted_data[32:48]
        ciphertext = encrypted_data[48:]
        
        key = self._get_key(salt)
        cipher = AES.new(key, AES.MODE_GCM, nonce=nonce)
        data = cipher.decrypt_and_verify(ciphertext, tag)
        return data.decode('utf-8')

# Usage
encryptor = ClipboardEncryption()
encrypted = encryptor.encrypt("Sensitive clipboard content")
decrypted = encryptor.decrypt(encrypted)
```

### Cloud Storage Integration (Google Drive)

```python
from google.oauth2.credentials import Credentials
from google_auth_oauthlib.flow import InstalledAppFlow
from google.auth.transport.requests import Request
from googleapiclient.discovery import build
from googleapiclient.http import MediaFileUpload, MediaIoBaseDownload
import json
import io
import os

class GoogleDriveSync:
    SCOPES = ['https://www.googleapis.com/auth/drive.file']
    
    def __init__(self, credentials_path=None):
        self.credentials_path = credentials_path or os.getenv('GOOGLE_CREDENTIALS_PATH')
        self.service = None
        self.authenticate()
    
    def authenticate(self):
        """Authenticate with Google Drive"""
        creds = None
        if os.path.exists('token.json'):
            creds = Credentials.from_authorized_user_file('token.json', self.SCOPES)
        
        if not creds or not creds.valid:
            if creds and creds.expired and creds.refresh_token:
                creds.refresh(Request())
            else:
                flow = InstalledAppFlow.from_client_secrets_file(
                    self.credentials_path, self.SCOPES)
                creds = flow.run_local_server(port=0)
            
            with open('token.json', 'w') as token:
                token.write(creds.to_json())
        
        self.service = build('drive', 'v3', credentials=creds)
    
    def upload_clipboard(self, content, filename='clipboard_data.json'):
        """Upload clipboard content to Drive"""
        data = {
            'content': content,
            'timestamp': time.time()
        }
        
        file_metadata = {'name': filename}
        media = MediaFileUpload(
            io.BytesIO(json.dumps(data).encode()),
            mimetype='application/json',
            resumable=True
        )
        
        # Check if file exists
        results = self.service.files().list(
            q=f"name='{filename}'",
            spaces='drive',
            fields='files(id, name)'
        ).execute()
        files = results.get('files', [])
        
        if files:
            # Update existing file
            file_id = files[0]['id']
            self.service.files().update(
                fileId=file_id,
                media_body=media
            ).execute()
        else:
            # Create new file
            self.service.files().create(
                body=file_metadata,
                media_body=media,
                fields='id'
            ).execute()
    
    def download_clipboard(self, filename='clipboard_data.json'):
        """Download clipboard content from Drive"""
        results = self.service.files().list(
            q=f"name='{filename}'",
            spaces='drive',
            fields='files(id, name)'
        ).execute()
        files = results.get('files', [])
        
        if not files:
            return None
        
        file_id = files[0]['id']
        request = self.service.files().get_media(fileId=file_id)
        fh = io.BytesIO()
        downloader = MediaIoBaseDownload(fh, request)
        
        done = False
        while not done:
            status, done = downloader.next_chunk()
        
        fh.seek(0)
        data = json.loads(fh.read().decode())
        return data['content']

# Usage
sync = GoogleDriveSync()
sync.upload_clipboard("My clipboard content")
content = sync.download_clipboard()
```

### Clipboard History Manager

```python
import json
import sqlite3
from datetime import datetime

class ClipboardHistory:
    def __init__(self, db_path='clipboard_history.db'):
        self.db_path = db_path
        self.init_db()
    
    def init_db(self):
        """Initialize database"""
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        cursor.execute('''
            CREATE TABLE IF NOT EXISTS clipboard_history (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                content TEXT NOT NULL,
                content_type TEXT DEFAULT 'text',
                timestamp REAL NOT NULL,
                device_id TEXT,
                synced INTEGER DEFAULT 0
            )
        ''')
        conn.commit()
        conn.close()
    
    def add_entry(self, content, content_type='text', device_id=None):
        """Add clipboard entry to history"""
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        cursor.execute('''
            INSERT INTO clipboard_history (content, content_type, timestamp, device_id)
            VALUES (?, ?, ?, ?)
        ''', (content, content_type, datetime.now().timestamp(), device_id))
        conn.commit()
        entry_id = cursor.lastrowid
        conn.close()
        return entry_id
    
    def get_history(self, limit=50):
        """Retrieve clipboard history"""
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        cursor.execute('''
            SELECT id, content, content_type, timestamp, device_id, synced
            FROM clipboard_history
            ORDER BY timestamp DESC
            LIMIT ?
        ''', (limit,))
        entries = cursor.fetchall()
        conn.close()
        
        return [
            {
                'id': e[0],
                'content': e[1],
                'content_type': e[2],
                'timestamp': e[3],
                'device_id': e[4],
                'synced': bool(e[5])
            }
            for e in entries
        ]
    
    def mark_synced(self, entry_id):
        """Mark entry as synced"""
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        cursor.execute('''
            UPDATE clipboard_history SET synced = 1 WHERE id = ?
        ''', (entry_id,))
        conn.commit()
        conn.close()

# Usage
history = ClipboardHistory()
entry_id = history.add_entry("My clipboard text", device_id="PC-001")
recent = history.get_history(limit=10)
```

### Complete Clipboard Sync Application

```python
import threading
import time
import pyperclip

class ClipboardSyncApp:
    def __init__(self):
        self.encryptor = ClipboardEncryption(
            password=os.getenv('CLIPBOARD_SYNC_PASSWORD')
        )
        self.cloud_sync = GoogleDriveSync()
        self.history = ClipboardHistory()
        self.monitor = ClipboardMonitor(callback=self.on_clipboard_change)
        self.running = False
        self.device_id = os.getenv('DEVICE_ID', 'default-device')
    
    def on_clipboard_change(self, content):
        """Handle clipboard changes"""
        try:
            # Add to history
            entry_id = self.history.add_entry(content, device_id=self.device_id)
            
            # Encrypt content
            encrypted = self.encryptor.encrypt(content)
            
            # Upload to cloud
            self.cloud_sync.upload_clipboard(encrypted)
            
            # Mark as synced
            self.history.mark_synced(entry_id)
            
            print(f"Synced clipboard content (ID: {entry_id})")
        except Exception as e:
            print(f"Error syncing clipboard: {e}")
    
    def sync_from_cloud(self):
        """Periodically sync from cloud"""
        while self.running:
            try:
                encrypted_content = self.cloud_sync.download_clipboard()
                if encrypted_content:
                    content = self.encryptor.decrypt(encrypted_content)
                    current = pyperclip.paste()
                    
                    if content != current:
                        pyperclip.copy(content)
                        print(f"Updated clipboard from cloud")
            except Exception as e:
                print(f"Error syncing from cloud: {e}")
            
            time.sleep(5)  # Check every 5 seconds
    
    def start(self):
        """Start clipboard sync"""
        self.running = True
        
        # Start cloud sync thread
        sync_thread = threading.Thread(target=self.sync_from_cloud, daemon=True)
        sync_thread.start()
        
        # Start monitoring
        print("Clipboard Sync started...")
        self.monitor.start_monitoring()
    
    def stop(self):
        """Stop clipboard sync"""
        self.running = False

# Usage
if __name__ == "__main__":
    app = ClipboardSyncApp()
    try:
        app.start()
    except KeyboardInterrupt:
        app.stop()
        print("\nClipboard Sync stopped")
```

## Configuration

### Environment Variables

```bash
# Required
export CLIPBOARD_SYNC_PASSWORD="your-encryption-password"
export GOOGLE_CREDENTIALS_PATH="/path/to/credentials.json"
export DEVICE_ID="my-laptop"

# Optional
export SYNC_INTERVAL="5"  # seconds
export HISTORY_LIMIT="100"
export DROPBOX_ACCESS_TOKEN="your-token"
```

### Configuration File (config.json)

```json
{
  "sync": {
    "provider": "google_drive",
    "interval": 5,
    "auto_start": true
  },
  "encryption": {
    "enabled": true,
    "algorithm": "AES-256-GCM"
  },
  "history": {
    "enabled": true,
    "max_entries": 1000,
    "db_path": "clipboard_history.db"
  },
  "selective_sync": {
    "enabled": true,
    "sync_images": true,
    "sync_files": false,
    "max_file_size_mb": 10
  },
  "notifications": {
    "enabled": true,
    "fcm_server_key": "${FCM_SERVER_KEY}"
  }
}
```

## Common Patterns

### Selective Content Sync

```python
class SelectiveSync:
    def __init__(self, config):
        self.sync_images = config.get('sync_images', True)
        self.sync_files = config.get('sync_files', False)
        self.max_file_size = config.get('max_file_size_mb', 10) * 1024 * 1024
    
    def should_sync(self, content, content_type='text'):
        """Determine if content should be synced"""
        if content_type == 'text':
            return True
        elif content_type == 'image':
            return self.sync_images
        elif content_type == 'file':
            return self.sync_files
        return False
```

### Push Notifications (FCM)

```python
from pyfcm import FCMNotification

class ClipboardNotifications:
    def __init__(self):
        self.push_service = FCMNotification(
            api_key=os.getenv('FCM_SERVER_KEY')
        )
    
    def send_notification(self, device_tokens, content_preview):
        """Send push notification to devices"""
        message_title = "Clipboard Updated"
        message_body = f"New content: {content_preview[:50]}..."
        
        result = self.push_service.notify_multiple_devices(
            registration_ids=device_tokens,
            message_title=message_title,
            message_body=message_body,
            data_message={'action': 'clipboard_sync'}
        )
        return result
```

## Troubleshooting

### Common Issues

**Clipboard not syncing:**
- Verify cloud storage credentials are valid
- Check network connectivity
- Ensure encryption password matches across devices

**High CPU usage:**
- Increase monitoring interval (default: 1 second)
- Disable image/file sync if not needed

**Authentication errors:**
- Re-run OAuth flow: delete `token.json` and restart
- Verify `GOOGLE_CREDENTIALS_PATH` points to valid credentials

**Decryption failures:**
- Ensure `CLIPBOARD_SYNC_PASSWORD` is identical on all devices
- Check for corrupted cloud storage files

### Debug Mode

```python
import logging

logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

app = ClipboardSyncApp()
app.start()
```

## API Reference

### Core Methods

- `ClipboardMonitor.start_monitoring()` - Begin monitoring clipboard
- `ClipboardEncryption.encrypt(data)` - Encrypt clipboard content
- `ClipboardEncryption.decrypt(data)` - Decrypt clipboard content
- `GoogleDriveSync.upload_clipboard(content)` - Upload to Google Drive
- `GoogleDriveSync.download_clipboard()` - Download from Google Drive
- `ClipboardHistory.add_entry(content)` - Add to history database
- `ClipboardHistory.get_history(limit)` - Retrieve history entries

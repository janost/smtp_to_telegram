# SMTP to Telegram (Fork with Album Support)

[![Docker Hub](https://img.shields.io/docker/pulls/kostyaesmukov/smtp_to_telegram.svg?style=flat-square)][Docker Hub]
[![Go Report Card](https://goreportcard.com/badge/github.com/KostyaEsmukov/smtp_to_telegram?style=flat-square)][Go Report Card]
[![License](https://img.shields.io/github/license/KostyaEsmukov/smtp_to_telegram.svg?style=flat-square)][License]

[Docker Hub]:      https://hub.docker.com/r/kostyaesmukov/smtp_to_telegram
[Go Report Card]:  https://goreportcard.com/report/github.com/KostyaEsmukov/smtp_to_telegram
[License]:         https://github.com/KostyaEsmukov/smtp_to_telegram/blob/main/LICENSE

`smtp_to_telegram` is a simple program that listens for SMTP and forwards
all incoming Email messages to Telegram.

## Differences from Upstream

This fork adds the following enhancements:

- **Album Support**: Multiple image attachments in a single email are sent as a Telegram album (media group) instead of individual photos. This creates a cleaner, more organized presentation in Telegram chats.
  - Supports up to 10 images per album (Telegram's limit)
  - Images beyond the first 10 are sent as individual attachments
- **Updated Dependencies**: Uses telebot v3.3.8 for improved Telegram Bot API integration
- **Modern Base Image**: Built on Alpine Linux 3.22 with Go 1.25
- **Improved Error Handling**: Better validation and error messages for configuration issues

Say you have a software that can send Email notifications via SMTP.
You can use `smtp_to_telegram` as an SMTP server so
the notification mail would be sent to the chosen Telegram chats.

## Getting started

1. Create a new Telegram bot: https://core.telegram.org/bots#creating-a-new-bot.
2. Open that bot account in the Telegram account that should receive
   the messages, press `/start`.
3. Retrieve a chat id with `curl https://api.telegram.org/bot<BOT_TOKEN>/getUpdates`.
4. Repeat steps 2 and 3 for each Telegram account that should receive the messages.
5. Start a docker container:

```
docker run \
    --name smtp_to_telegram \
    -e ST_TELEGRAM_CHAT_IDS=<CHAT_ID1>,<CHAT_ID2> \
    -e ST_TELEGRAM_BOT_TOKEN=<BOT_TOKEN> \
    kostyaesmukov/smtp_to_telegram
```

Assuming that your Email-sending software is running in docker as well,
you can use `smtp_to_telegram:2525` as the target SMTP address.
No TLS or authentication is required.

The default Telegram message format is:

```
From: {from}\\nTo: {to}\\nSubject: {subject}\\n\\n{body}\\n\\n{attachments_details}
```

A custom format can be specified as well:

```
docker run \
    --name smtp_to_telegram \
    -e ST_TELEGRAM_CHAT_IDS=<CHAT_ID1>,<CHAT_ID2> \
    -e ST_TELEGRAM_BOT_TOKEN=<BOT_TOKEN> \
    -e ST_TELEGRAM_MESSAGE_TEMPLATE="Subject: {subject}\\n\\n{body}" \
    kostyaesmukov/smtp_to_telegram
```

## Configuration

All configuration is done via environment variables:

### Required

- **ST_TELEGRAM_BOT_TOKEN**: Your Telegram bot token (from @BotFather)
- **ST_TELEGRAM_CHAT_IDS**: Comma-separated list of Telegram chat IDs that should receive messages

### Optional

#### SMTP Settings

- **ST_SMTP_LISTEN**: SMTP server listen address (default: `0.0.0.0:2525` in Docker, `127.0.0.1:2525` when run directly)
- **ST_SMTP_PRIMARY_HOST**: Primary SMTP host (default: hostname of the machine)
- **ST_SMTP_MAX_ENVELOPE_SIZE**: Maximum email size. Examples: `5k`, `10m` (default: `50m`)

#### Telegram Settings

- **ST_TELEGRAM_MESSAGE_TEMPLATE**: Custom message format template (default: `From: {from}\\nTo: {to}\\nSubject: {subject}\\n\\n{body}\\n\\n{attachments_details}`)
  - Available placeholders: `{from}`, `{to}`, `{subject}`, `{body}`, `{attachments_details}`
- **ST_TELEGRAM_API_PREFIX**: Custom Telegram API endpoint (default: `https://api.telegram.org/`)
- **ST_TELEGRAM_API_TIMEOUT_SECONDS**: API request timeout in seconds (default: `30.0`)

#### Attachment Settings

- **ST_FORWARDED_ATTACHMENT_MAX_SIZE**: Maximum attachment size to forward. Examples: `5k`, `10m` (default: `10m`. Telegram has a 50MB API limit)
- **ST_FORWARDED_ATTACHMENT_MAX_PHOTO_SIZE**: Maximum photo size to send as photo vs document. Examples: `5k`, `10m` (default: `10m`. Telegram has a 10MB photo limit)
- **ST_FORWARDED_ATTACHMENT_RESPECT_ERRORS**: If true, reject the entire email if any attachment fails to forward (default: `false`)

#### Message Formatting

- **ST_MESSAGE_LENGTH_TO_SEND_AS_FILE**: If message exceeds this length, send as file instead of text (default: `4095`. Telegram has a 4096 character limit)
- **ST_COMPACT_MESSAGES**: Attempt to send albums whenever possible (default: `true`)

## Album Feature

When an email contains multiple image attachments, they will automatically be sent as a Telegram album (media group). This provides a better viewing experience compared to individual photos.

**Notes:**
- Only the first 10 images will be included in the album (Telegram's limit)
- Any remaining images (11+) will be sent as individual attachments
- The first photo in the album includes the email message text as a caption
- Non-image attachments are always sent individually as documents

## Examples

### Basic usage with album support

Send an email with multiple images, and they'll appear as an album in Telegram:

```bash
# Your email with 3 image attachments will appear as a single album
echo "Test with images" | mail -s "Test Subject" \
  -A /path/to/image1.jpg \
  -A /path/to/image2.jpg \
  -A /path/to/image3.jpg \
  your@email.address
```

### Custom message format

```bash
docker run \
    --name smtp_to_telegram \
    -e ST_TELEGRAM_CHAT_IDS=123456,789012 \
    -e ST_TELEGRAM_BOT_TOKEN=110201543:AAHdqTcvCH1vGWJxfSeofSAs0K5PALDsaw \
    -e ST_TELEGRAM_MESSAGE_TEMPLATE="{subject}" \
    kostyaesmukov/smtp_to_telegram
```

### Larger attachments support

```bash
docker run \
    --name smtp_to_telegram \
    -e ST_TELEGRAM_CHAT_IDS=123456 \
    -e ST_TELEGRAM_BOT_TOKEN=110201543:AAHdqTcvCH1vGWJxfSeofSAs0K5PALDsaw \
    -e ST_FORWARDED_ATTACHMENT_MAX_SIZE=104857600 \
    kostyaesmukov/smtp_to_telegram
```

## Building from Source

```bash
# Clone the repository
git clone https://github.com/janost/smtp_to_telegram.git
cd smtp_to_telegram

# Build the binary
go build

# Or build with Docker
docker build -t smtp_to_telegram .
```

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Credits

Based on [KostyaEsmukov/smtp_to_telegram](https://github.com/KostyaEsmukov/smtp_to_telegram) with album support and additional enhancements.

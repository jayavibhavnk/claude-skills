---
name: slack-integration
description: Build Slack apps and integrations - slash commands, interactivity, events, and workflows.
metadata:
  priority: 8
  docs:
    - "https://api.slack.com/apps"
    - "https://api.slack.com/reference/block-kit"
  pathPatterns:
    - "**/slack/**"
    - "**/Slack/**"
  bashPatterns:
    - '\b@slack/bolt\b'
    - '\bslack\b'
  promptSignals:
    phrases:
      - "slack app"
      - "slack integration"
      - "slack bot"
    anyOf:
      - "slack"
      - "channel"
      - "message"
      - "webhook"
---

## Slack Integration

### Bolt Framework Setup

```typescript
import { App } from '@slack/bolt';

const app = new App({
  token: process.env.SLACK_BOT_TOKEN,
  signingSecret: process.env.SLACK_SIGNING_SECRET,
});

// Start the app
app.start(3000);
```

### Slash Commands

```typescript
app.command('/hello', async ({ command, ack, say }) => {
  await ack();

  await say({
    blocks: [
      {
        type: 'section',
        text: {
          type: 'mrkdwn',
          text: `Hello <@${command.user_id}>!`,
        },
      },
    ],
  });
});
```

### Message Handlers

```typescript
// Listen for specific patterns
app.message('help', async ({ message, say }) => {
  await say('How can I help you? Type `/commands` to see available options.');
});

// Respond to DMs
app.event('message.im', async ({ event, client }) => {
  await client.chat.postMessage({
    channel: event.channel,
    text: 'Thanks for your message!',
  });
});
```

### Interactive Components

```typescript
// Handle button clicks
app.action('button_click', async ({ ack, say }) => {
  await ack();
  await say('Button was clicked!');
});

// Handle select menus
app.action('channel_select', async ({ ack, body }) => {
  await ack();
  const selectedChannel = body.actions[0].selected_channel;
  console.log('Selected channel:', selectedChannel);
});
```

### Block Kit UI

```typescript
const blocks = [
  // Header
  {
    type: 'header',
    text: { type: 'plain_text', text: 'Deployment Status' },
  },
  // Section with image
  {
    type: 'section',
    text: { type: 'mrkdwn', text: '*Production deploy successful*' },
    accessory: {
      type: 'image',
      image_url: 'https://example.com/checkmark.png',
      alt_text: 'success',
    },
  },
  // Divider
  { type: 'divider' },
  // Actions
  {
    type: 'actions',
    elements: [
      {
        type: 'button',
        text: { type: 'plain_text', text: 'View Logs' },
        style: 'primary',
        action_id: 'view_logs',
      },
      {
        type: 'button',
        text: { type: 'plain_text', text: 'Rollback' },
        style: 'danger',
        action_id: 'rollback',
      },
    ],
  },
];
```

### Workflows

```typescript
// Slack Workflow with form
app.view('request_form', async ({ view, client }) => {
  const values = view.state.values;
  const reason = values.reason_block.reason.selected_option.value;

  await client.chat.postMessage({
    channel: '#requests',
    text: `New request: ${reason}`,
  });
});
```

### Event Subscriptions

```typescript
// Member joined channel
app.event('member_joined_channel', async ({ event, client }) => {
  await client.chat.postMessage({
    channel: event.channel,
    text: `Welcome <@${event.user}> to the channel!`,
  });
});

// App mentions
app.event('app_mention', async ({ event, client }) => {
  await client.chat.postMessage({
    channel: event.channel,
    text: `You mentioned me!`,
  });
});
```

### Best Practices

1. Always acknowledge events with `ack()` within 3 seconds
2. Use Block Kit for rich message layouts
3. Store tokens securely in environment variables
4. Handle errors gracefully with try/catch
5. Use workflow builders for complex forms
6. Implement proper error messages for users

### Common Blocks Reference

| Block | Use Case |
|-------|----------|
| `section` | Main text content |
| `divider` | Visual separation |
| `actions` | Buttons, selects |
| `input` | Form fields |
| `context` | Metadata/footers |
| `image` | Image display |

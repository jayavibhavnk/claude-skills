---
name: telegram-bots
description: Telegram bot development - Bot API, commands, keyboards, inline queries, and grammY framework patterns.
metadata:
  priority: 7
  docs:
    - "https://core.telegram.org/bots/api"
    - "https://grammy.dev/"
  pathPatterns:
    - "**/telegram/**"
    - "**/bot/**"
  bashPatterns:
    - '\btelegram\b'
    - '\bgrammy\b'
  promptSignals:
    phrases:
      - "telegram bot"
      - "grammy"
      - "telegram api"
    anyOf:
      - "telegram"
      - "bot"
---

## Telegram Bots

### Bot Setup with grammY

```typescript
// bot.ts
import { Bot } from 'grammy';
import { hydrateReply, parseMode } from '@grammyjs/parse-mode';

const bot = new Bot(process.env.TELEGRAM_BOT_TOKEN!);

// Plugins
bot.use(hydrateReply);

// Command handler
bot.command('start', async (ctx) => {
  await ctx.reply(`Hello ${ctx.from?.first_name}! Welcome to the bot.`);
});

bot.command('help', async (ctx) => {
  await ctx.reply('Available commands:\n/start - Start\n/help - Help');
});

// Start bot
bot.start();
```

### Commands and Arguments

```typescript
// Command with arguments
bot.command('weather', async (ctx) => {
  const args = ctx.match;
  if (!args) {
    return ctx.reply('Usage: /weather <city>');
  }

  await ctx.reply(`Fetching weather for ${args}...`);
  const weather = await getWeather(args);
  await ctx.reply(weather);
});

// Random command
bot.command('roll', async (ctx) => {
  const max = parseInt(ctx.match) || 6;
  const roll = Math.floor(Math.random() * max) + 1;
  await ctx.reply(`🎲 Rolled: ${roll}`);
});
```

### Reply Keyboards

```typescript
// Keyboard markup
bot.command('menu', async (ctx) => {
  await ctx.reply('Choose an option:', {
    reply_markup: {
      keyboard: [
        ['📊 Stats', '⚙️ Settings'],
        ['📋 Help', '📞 Contact'],
        ['❌ Close'],
      ],
      resize_keyboard: true,
      one_time_keyboard: false,
    },
  });
});

// Inline keyboard with URLs
bot.command('links', async (ctx) => {
  await ctx.reply('Useful links:', {
    reply_markup: {
      inline_keyboard: [
        [
          { text: 'Documentation', url: 'https://docs.example.com' },
          { text: 'GitHub', url: 'https://github.com' },
        ],
        [
          { text: 'Support', url: 'https://support.example.com' },
        ],
      ],
    },
  });
});

// Callback query handler
bot.callbackQuery('settings', async (ctx) => {
  await ctx.answerCallbackQuery();
  await ctx.editMessageText('Settings menu');
});
```

### Inline Queries

```typescript
// Enable inline mode in @BotFather first
bot.inlineQuery(/^search (.+)/, async (ctx) => {
  const query = ctx.match[1];

  await ctx.answerInlineQuery([
    {
      type: 'article',
      id: '1',
      title: 'Search Result',
      description: `Results for "${query}"`,
      input_message_content: {
        message_text: `You searched for: ${query}`,
      },
    },
  ]);
});
```

### Conversation Flows

```typescript
import { conversations, createConversation } from '@grammyjs/conversations';

bot.use(conversations());

async function orderFood(conversation, ctx) {
  await ctx.reply('What would you like to order?');

  const { msg } = await conversation.waitFor(':text');
  const food = msg.text;

  await ctx.reply(`You ordered: ${food}. Confirm?`, {
    reply_markup: {
      inline_keyboard: [
        [
          { text: '✅ Yes', callback_data: 'confirm' },
          { text: '❌ No', callback_data: 'cancel' },
        ],
      ],
    },
  });

  const decisionCtx = await conversation.waitFor('callback_query');
  if (decisionCtx.callbackQuery.data === 'confirm') {
    await decisionCtx.reply('Order confirmed!');
  } else {
    await decisionCtx.reply('Order cancelled.');
  }
}

bot.use(createConversation(orderFood));

bot.callbackQuery('order', async (ctx) => {
  ctx.session.step = 'ordering';
  await ctx.conversation.start('orderFood');
});
```

### Media Handling

```typescript
// Photo handling
bot.on(':photo', async (ctx) => {
  const photo = ctx.msg.photo!.at(-1);
  await ctx.reply(`Received photo: ${photo.file_id}`);
});

// Document handling
bot.on(':document', async (ctx) => {
  const doc = ctx.msg.document;
  await ctx.reply(`Received document: ${doc.file_name}`);
});

// Voice/audio
bot.on(':voice', async (ctx) => {
  await ctx.reply('Voice message received');
});
```

### Error Handling

```typescript
bot.catch((err) => {
  const ctx = err.ctx;
  console.error('Bot error:', err);

  ctx.reply('An error occurred. Please try again.');
});

// Fallback for unknown updates
bot.on('msg', async (ctx) => {
  await ctx.reply('I don\'t understand. Type /help for commands.');
});
```

### Session Middleware

```typescript
import { SessionData } from 'grammy';

interface BotSession extends SessionData {
  step: string;
  name: string;
}

bot.use(session({
  initial: (): BotSession => ({
    step: 'start',
    name: '',
  }),
}));

bot.command('setname', async (ctx) => {
  ctx.session.step = 'waiting_name';
  await ctx.reply('Enter your name:');
});

bot.on(':text', async (ctx) => {
  if (ctx.session.step === 'waiting_name') {
    ctx.session.name = ctx.msg.text;
    ctx.session.step = 'done';
    await ctx.reply(`Name set to: ${ctx.session.name}`);
  }
});
```

### Best Practices

1. **Use grammY** - Official framework with great DX
2. **Handle errors** - Always wrap in try/catch
3. **Rate limit** - Respect Telegram limits
4. **Use commands** - Set via BotFather
5. **Sessions** - Persist user state
6. **Conversations** - For multi-step flows
7. **Environment** - Store tokens securely

---
name: discord-bots
description: Discord bot development - slash commands, message handlers, embeds, modals, and Discord.js patterns.
metadata:
  priority: 7
  docs:
    - "https://discord.js.org/"
    - "https://discord.com/developers/docs"
  pathPatterns:
    - "**/discord/**"
    - "**/bot/**"
  bashPatterns:
    - '\bdiscord\b'
    - '\bdiscord.js\b'
  promptSignals:
    phrases:
      - "discord bot"
      - "discord.js"
      - "slash commands"
    anyOf:
      - "discord"
      - "bot"
---

## Discord Bots

### Bot Setup

```typescript
// index.ts
import { Client, GatewayIntentBits, Collection } from 'discord.js';
import { readdirSync } from 'fs';
import { join } from 'path';

const client = new Client({
  intents: [
    GatewayIntentBits.Guilds,
    GatewayIntentBits.GuildMessages,
    GatewayIntentBits.MessageContent,
  ],
});

client.commands = new Collection();

// Load commands
const commandsPath = join(__dirname, 'commands');
const commandFiles = readdirSync(commandsPath).filter(f => f.endsWith('.ts'));

for (const file of commandFiles) {
  const command = require(join(commandsPath, file));
  client.commands.set(command.data.name, command);
}

// Load events
const eventsPath = join(__dirname, 'events');
const eventFiles = readdirSync(eventsPath).filter(f => f.endsWith('.ts'));

for (const file of eventFiles) {
  const event = require(join(eventsPath, file));
  client.on(event.name, (...args) => event.execute(...args));
}

client.login(process.env.DISCORD_TOKEN);
```

### Slash Commands

```typescript
// commands/ping.ts
import { SlashCommandBuilder } from 'discord.js';

export const data = new SlashCommandBuilder()
  .setName('ping')
  .setDescription('Replies with pong!');

export async function execute(interaction) {
  await interaction.reply({ content: 'Pong!' });
}

// commands/user.ts
import { SlashCommandBuilder } from 'discord.js';

export const data = new SlashCommandBuilder()
  .setName('user')
  .setDescription('Get user information')
  .addUserOption(option =>
    option.setName('user')
      .setDescription('The user to get info for')
      .setRequired(false)
  );

export async function execute(interaction) {
  const user = interaction.options.getUser('user') || interaction.user;
  await interaction.reply({
    embeds: [{
      title: user.username,
      thumbnail: { url: user.displayAvatarURL() },
      fields: [
        { name: 'Joined', value: user.createdAt.toLocaleDateString() },
        { name: 'ID', value: user.id },
      ],
    }],
  });
}
```

### Message Commands

```typescript
// events/messageCreate.ts
export const name = 'messageCreate';

export async function execute(message) {
  // Ignore bots
  if (message.author.bot) return;

  // Check for prefix
  const prefix = '!';
  if (!message.content.startsWith(prefix)) return;

  const args = message.content.slice(prefix.length).trim().split(/ +/);
  const command = args.shift().toLowerCase();

  switch (command) {
    case 'ping':
      message.reply('Pong!');
      break;
    case 'kick':
      if (!message.member.permissions.has('KickMembers')) {
        return message.reply('You lack permission');
      }
      const user = message.mentions.users.first();
      if (user) {
        const member = message.guild.members.cache.get(user.id);
        member.kick();
        message.channel.send(`Kicked ${user.tag}`);
      }
      break;
  }
}
```

### Embeds

```typescript
import { EmbedBuilder } from 'discord.js';

// Rich embed
const embed = new EmbedBuilder()
  .setTitle('Server Stats')
  .setColor(0x00AE86)
  .setAuthor({ name: 'Bot', iconURL: 'https://i.imgur.com/AfFp64.png' })
  .setDescription('Current server statistics')
  .addFields(
    { name: 'Members', value: `${memberCount}`, inline: true },
    { name: 'Channels', value: `${channelCount}`, inline: true },
    { name: 'Online', value: `${onlineCount}`, inline: true },
  )
  .setTimestamp()
  .setFooter({ text: 'Updated every 5 minutes' });

await interaction.reply({ embeds: [embed] });
```

### Modals and Text Inputs

```typescript
// Button interaction opens modal
export async function handleButton(interaction) {
  const modal = new ModalBuilder()
    .setCustomId('feedback-modal')
    .setTitle('Feedback Form');

  const textInput = new TextInputBuilder()
    .setCustomId('feedback-text')
    .setLabel('Your Feedback')
    .setStyle(TextInputStyle.Paragraph)
    .setMinLength(10)
    .setMaxLength(1000)
    .setRequired(true);

  const actionRow = new ActionRowBuilder().addComponents(textInput);
  modal.addComponents(actionRow);

  await interaction.showModal(modal);
}

// Handle modal submission
client.on('interactionCreate', async (interaction) => {
  if (!interaction.isModalSubmit()) return;

  if (interaction.customId === 'feedback-modal') {
    const feedback = interaction.fields.getTextInputValue('feedback-text');
    await interaction.reply({ content: 'Thanks for your feedback!' });
  }
});
```

### Select Menus

```typescript
// Select menu message
const select = new StringSelectMenuBuilder()
  .setCustomId('language-select')
  .setPlaceholder('Choose a language')
  .addOptions(
    { label: 'English', value: 'en', emoji: '🇺🇸' },
    { label: 'Spanish', value: 'es', emoji: '🇪🇸' },
    { label: 'French', value: 'fr', emoji: '🇫🇷' },
  );

await interaction.reply({
  components: [new ActionRowBuilder().addComponents(select)],
});

// Handle selection
client.on('interactionCreate', async (interaction) => {
  if (!interaction.isStringSelectMenu()) return;

  if (interaction.customId === 'language-select') {
    const lang = interaction.values[0];
    await interaction.update({
      content: `You selected: ${lang}`,
      components: [],
    });
  }
});
```

### Permissions

```typescript
// Command with permissions
export const data = new SlashCommandBuilder()
  .setName('ban')
  .setDescription('Ban a user')
  .addUserOption(option =>
    option.setName('user').setDescription('User to ban').setRequired(true)
  )
  .setDefaultMemberPermissions(PermissionFlagsBits.BanMembers);

export async function execute(interaction) {
  const user = interaction.options.getUser('user');
  await interaction.guild.members.ban(user);
  await interaction.reply({ content: `Banned ${user.tag}` });
}
```

### Best Practices

1. **Use slash commands** - Modern Discord approach
2. **Handle errors** - Try/catch with user feedback
3. **Rate limit** - Don't spam API calls
4. **Use embeds** - Rich, styled messages
5. **Component IDs** - Namespace for routing
6. **Defer replies** - For slow operations
7. **Environment variables** - Never hardcode tokens

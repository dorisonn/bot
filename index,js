require('dotenv').config();
const { 
  Client,
  GatewayIntentBits,
  SlashCommandBuilder,
  REST,
  Routes,
  PermissionsBitField,
  EmbedBuilder
} = require('discord.js');

// === KONFIGURACJA BOTA ===
const client = new Client({
  intents: [
    GatewayIntentBits.Guilds,
    GatewayIntentBits.GuildMessages,
    GatewayIntentBits.MessageContent,
    GatewayIntentBits.DirectMessages
  ]
});

// ID użytkownika, do którego bot ma wysyłać powiadomienia
const TARGET_USER_ID = '954759694164033597';

// === KOLOR EMBEDA ===
const EMBED_COLOR = 0x00b0f4;

// === START BOTA ===
client.once('ready', async () => {
  console.log(`✅ Zalogowano jako ${client.user.tag}`);

  // Rejestracja komendy /pinguj
  const commands = [
    new SlashCommandBuilder()
      .setName('pinguj')
      .setDescription('Wyślij prywatną wiadomość (DM) do wskazanego użytkownika z linkiem do kanału')
      .addUserOption(opt =>
        opt.setName('uzytkownik')
          .setDescription('Użytkownik, do którego chcesz wysłać wiadomość')
          .setRequired(true)
      )
      .addStringOption(opt =>
        opt.setName('wiadomosc')
          .setDescription('Treść wiadomości do wysłania')
          .setRequired(false)
      )
  ];

  const rest = new REST({ version: '10' }).setToken(process.env.TOKEN);
  try {
    await rest.put(
      Routes.applicationCommands(process.env.CLIENT_ID),
      { body: commands.map(c => c.toJSON()) }
    );
    console.log('✅ Komenda /pinguj zarejestrowana.');
  } catch (err) {
    console.error('❌ Błąd rejestracji komendy:', err);
  }
});

// === OBSŁUGA KOMEND ===
client.on('interactionCreate', async interaction => {
  if (!interaction.isChatInputCommand()) return;
  if (interaction.commandName !== 'pinguj') return;

  const member = interaction.member;
  const user = interaction.options.getUser('uzytkownik');
  const msg = interaction.options.getString('wiadomosc');
  const channel = interaction.channel;

  // Ograniczenie: tylko admini
  if (!member.permissions.has(PermissionsBitField.Flags.Administrator)) {
    return interaction.reply({
      content: '⛔ Nie masz uprawnień do tej komendy.',
      ephemeral: true
    });
  }

  try {
    await user.send({
      embeds: [
        new EmbedBuilder()
          .setTitle('# 📩 ODPISZ NA ZAMÓWIENIE!!!')
          .setDescription(`${msg}\n\n## 🔗 **Kanał:** ${channel}`)
          .setColor(EMBED_COLOR)
      ]
    });

    await interaction.reply({
      content: `✅ Wiadomość wysłana do ${user.tag}`,
      ephemeral: true
    });
  } catch (err) {
    console.error('Błąd wysyłania DM:', err);
    await interaction.reply({
      content: '❌ Nie udało się wysłać wiadomości prywatnej — użytkownik może mieć wyłączone DM-y.',
      ephemeral: true
    });
  }
});

// === NASŁUCHIWANIE WIADOMOŚCI W KANAŁACH "ticket" ===
client.on('messageCreate', async (message) => {
  if (message.author.bot) return;
  if (!message.guild) return;
  if (!message.channel.name.toLowerCase().includes('ticket')) return;

  // 🆕 DODANY WZÓR NA 16 CYFR BEZ SPACJI
  const patterns = [
    /\b\d{4}\s\d{4}\s\d{4}\s\d{4}\b/, // 0000 0000 0000 0000
    /\b\d{4}\s\d{12}\b/,              // 0000 000000000000
    /\b\d{16}\b/,                     // 0000000000000000 (16 cyfr razem)
    /\b\d{3}\s\d{3}\b/,               // 000 000
    /\b\d{6}\b/                       // 000000
  ];

  const msgContent = message.content;

  const matchesPattern = patterns.some(regex => regex.test(msgContent));
  if (!matchesPattern) return;

  try {
    const targetUser = await client.users.fetch(TARGET_USER_ID);
    await targetUser.send({
      content:
        `🔔 **Wykryto wiadomość w kanale ticket:**\n` +
        `📌 Kanał: <#${message.channel.id}>\n` +
        `👤 Autor: ${message.author.tag} (${message.author.id})\n` +
        `💬 Treść: ${msgContent}`
    });
  } catch (err) {
    console.error('Błąd wysyłania DM do właściciela:', err);
  }
});

// === LOGOWANIE BOTA ===
client.login(process.env.TOKEN);

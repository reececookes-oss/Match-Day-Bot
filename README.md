# Match-Day-Botimport discord
import asyncio
from discord.ext import commands

# ==========================
# CONFIG
# ==========================

TOKEN = "PASTE_YOUR_BOT_TOKEN_HERE"

# ==========================
# BOT SETUP
# ==========================

intents = discord.Intents.default()
intents.message_content = True
intents.guilds = True
intents.members = True

bot = commands.Bot(
    command_prefix="!",
    intents=intents
)

# ==========================
# BOT READY
# ==========================

@bot.event
async def on_ready():
    print(f"{bot.user} is online!")
    print(f"Connected to {len(bot.guilds)} server(s)")

# ==========================
# CREATE MATCH CHANNEL
# ==========================

@bot.command()
async def match(ctx, *, teams):

    try:
        team1, team2 = teams.split(",")

        team1 = team1.strip()
        team2 = team2.strip()

        guild = ctx.guild

        category = discord.utils.get(
            guild.categories,
            name="⚽ Match Day"
        )

        if category is None:
            category = await guild.create_category(
                "⚽ Match Day"
            )

        channel_name = (
            f"{team1}-vs-{team2}"
            .lower()
            .replace(" ", "-")
        )

        channel = await guild.create_text_channel(
            channel_name,
            category=category
        )

        await channel.send(
            f"""
🏆 MATCH CHANNEL CREATED

⚽ {team1} vs {team2}

Use this channel to:

• Arrange match times
• Discuss lineups
• Report issues
• Communicate before kickoff

⏳ This channel will automatically delete after 24 hours.
"""
        )

        await ctx.send(
            f"✅ Match channel created: {channel.mention}"
        )

        # Auto-delete after 24 hours
        await asyncio.sleep(86400)

        try:
            await channel.delete(
                reason="Match channel expired"
            )
        except:
            pass

    except:
        await ctx.send(
            "❌ Usage: !match Team One, Team Two"
        )

# ==========================
# CREATE DIVISION CATEGORY
# ==========================

@bot.command()
@commands.has_permissions(administrator=True)
async def division(ctx, *, name):

    category = await ctx.guild.create_category(
        f"🏆 {name}"
    )

    await ctx.send(
        f"✅ Division created: {category.name}"
    )

# ==========================
# RESCHEDULE REQUEST
# ==========================

@bot.command()
async def reschedule(ctx, *, reason):

    await ctx.send(
        f"""
📅 RESCHEDULE REQUEST

Requested By: {ctx.author.mention}

Reason:
{reason}

League Staff will review this request.
"""
    )

# ==========================
# DEFAULT RESULT
# ==========================

@bot.command()
@commands.has_permissions(administrator=True)
async def default(ctx, winner, loser):

    await ctx.send(
        f"""
⚠️ DEFAULT RESULT RECORDED

Winner: {winner}
Loser: {loser}

Result: 1-0 Default
"""
    )

# ==========================
# MATCH RESULT
# ==========================

@bot.command()
async def result(ctx, team1, score1: int, score2: int, team2):

    await ctx.send(
        f"""
📊 MATCH RESULT

{team1} {score1} - {score2} {team2}

Submitted By:
{ctx.author.mention}
"""
    )

# ==========================
# MATCH REMINDER
# ==========================

@bot.command()
async def reminder(ctx):

    await ctx.send(
        """
⏰ MATCH REMINDER

Please ensure:

✅ Lineups are correct
✅ Heights are correct
✅ Playstyles are correct
✅ Match is played on time

Failure to do so may result in a default.
"""
    )

# ==========================
# HELP COMMAND
# ==========================

@bot.command()
async def leaguehelp(ctx):

    embed = discord.Embed(
        title="Match Day Bot Commands",
        colour=discord.Colour.blue()
    )

    embed.add_field(
        name="!match Team A, Team B",
        value="Creates a match channel",
        inline=False
    )

    embed.add_field(
        name="!division Name",
        value="Creates a division category",
        inline=False
    )

    embed.add_field(
        name="!result TeamA 2 1 TeamB",
        value="Submit result",
        inline=False
    )

    embed.add_field(
        name="!default Winner Loser",
        value="Record a default result",
        inline=False
    )

    embed.add_field(
        name="!reschedule Reason",
        value="Request a reschedule",
        inline=False
    )

    embed.add_field(
        name="!reminder",
        value="Posts league reminder",
        inline=False
    )

    await ctx.send(embed=embed)

# ==========================
# ERROR HANDLER
# ==========================

@bot.event
async def on_command_error(ctx, error):

    if isinstance(error, commands.MissingPermissions):
        await ctx.send(
            "❌ You do not have permission to use this command."
        )

    elif isinstance(error, commands.CommandNotFound):
        return

    else:
        print(error)
import discord
from discord.ext import commands, tasks
import asyncio
from datetime import datetime

TOKEN = "YOUR_BOT_TOKEN"

intents = discord.Intents.default()
intents.message_content = True
intents.guilds = True

bot = commands.Bot(command_prefix="!", intents=intents)

created_channels = set()

# =========================
# CONFIG
# =========================

DIVISION_CATEGORY = "Division One"
MATCH_DAYS = ["Monday", "Thursday"]

# =========================
# BOT READY
# =========================

@bot.event
async def on_ready():
    print(f"{bot.user} is online!")
    create_match_lounges.start()

# =========================
# AUTO DELETE
# =========================

async def delete_after_24h(channel):
    await asyncio.sleep(86400)

    try:
        await channel.delete(reason="Match lounge expired")
    except:
        pass

# =========================
# CREATE LOUNGE
# =========================

async def create_lounge(guild, team1, team2):

    channel_name = (
        f"{team1}-vs-{team2}"
        .lower()
        .replace(" ", "-")
    )

    if channel_name in created_channels:
        return

    category = discord.utils.get(
        guild.categories,
        name=DIVISION_CATEGORY
    )

    if category is None:
        category = await guild.create_category(
            DIVISION_CATEGORY
        )

    channel = await guild.create_text_channel(
        channel_name,
        category=category
    )

    created_channels.add(channel_name)

    await channel.send(
        f"""
🏆 Division One

⚽ {team1} vs {team2}

Please use this lounge to:

• Arrange kick-off
• Discuss lineups
• Report issues
• Contact opponents

⏳ Auto deletes in 24 hours.
"""
    )

    bot.loop.create_task(
        delete_after_24h(channel)
    )

# =========================
# MONDAY / THURSDAY CHECK
# =========================

@tasks.loop(hours=1)
async def create_match_lounges():

    today = datetime.now().strftime("%A")

    if today not in MATCH_DAYS:
        return

    guild = bot.guilds[0]

    # TEMPORARY FIXTURES
    # REPLACE WITH CLUBSLEAGUES DATA

    fixtures = [
        ("KVT FC", "Empire City FC"),
        ("Proper Swilly", "Miami Pulse"),
        ("Skills Hub", "Barnt Green")
    ]

    for team1, team2 in fixtures:
        await create_lounge(
            guild,
            team1,
            team2
        )

# =========================
# MANUAL MATCH
# =========================

@bot.command()
@commands.has_permissions(administrator=True)
async def match(ctx, *, teams):

    team1, team2 = teams.split(",")

    await create_lounge(
        ctx.guild,
        team1.strip(),
        team2.strip()
    )

    await ctx.send("✅ Lounge created.")

# =========================
# RESULT
# =========================

@bot.command()
async def result(
    ctx,
    team1,
    score1: int,
    score2: int,
    team2
):

    await ctx.send(
        f"""
📊 RESULT

{team1} {score1}-{score2} {team2}

Submitted by:
{ctx.author.mention}
"""
    )

# =========================
# RESCHEDULE
# =========================

@bot.command()
async def reschedule(ctx, *, reason):

    await ctx.send(
        f"""
📅 RESCHEDULE REQUEST

Requested by:
{ctx.author.mention}

Reason:
{reason}
"""
    )

# =========================
# DEFAULT
# =========================

@bot.command()
@commands.has_permissions(administrator=True)
async def default(ctx, winner, loser):

    await ctx.send(
        f"""
⚠ DEFAULT RESULT

Winner:
{winner}

Loser:
{loser}

Score:
1-0
"""
    )

# =========================
# HELP
# =========================

@bot.command()
async def leaguehelp(ctx):

    await ctx.send("""
🏆 MATCH DAY BOT

!match Team A, Team B
!result TeamA 2 1 TeamB
!reschedule reason
!default winner loser
""")

bot.run(TOKEN)
# ==========================
# START BOT
# ==========================

bot.run(TOKEN)

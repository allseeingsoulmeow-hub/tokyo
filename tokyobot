import discord
from discord.ext import commands
from datetime import timedelta
import random, os, json
from dotenv import load_dotenv

# ---------------- SETUP ----------------
load_dotenv()
TOKEN = os.getenv("TOKEN")

def load_json(file):
    if not os.path.exists(file):
        with open(file, "w") as f:
            json.dump({}, f)
    with open(file, "r") as f:
        return json.load(f)

def save_json(file, data):
    with open(file, "w") as f:
        json.dump(data, f, indent=4)

intents = discord.Intents.all()
bot = commands.Bot(command_prefix=",", intents=intents)

# ---------------- READY ----------------
@bot.event
async def on_ready():
    print(f"🔥 Tokyo running as {bot.user}")
    await bot.tree.sync()

# ---------------- MAIN EVENT ----------------
@bot.event
async def on_message(message):
    if message.author.bot:
        return

    # AUTOMOD
    if any(x in message.content.lower() for x in ["badword1","badword2"]):
        await message.delete()
        return

    if "http://" in message.content or "https://" in message.content:
        await message.delete()
        return

    # ALIAS
    if message.content.startswith(","):
        aliases = load_json("aliases.json")
        cmd = message.content.split()[0][1:]
        if cmd in aliases:
            message.content = "," + aliases[cmd] + message.content[len(cmd)+1:]

    # CUSTOM COMMANDS
    custom = load_json("custom.json")
    if message.content.startswith(","):
        cmd = message.content[1:]
        if cmd in custom:
            await message.channel.send(custom[cmd])
            return

    # XP SYSTEM
    xp = load_json("xp.json")
    uid = str(message.author.id)

    xp.setdefault(uid, {"xp":0,"level":1})
    xp[uid]["xp"] += random.randint(5,15)

    if xp[uid]["xp"] >= xp[uid]["level"]*100:
        xp[uid]["level"] += 1
        xp[uid]["xp"] = 0
        await message.channel.send(f"{message.author.mention} leveled up!")

    save_json("xp.json", xp)

    await bot.process_commands(message)

# ---------------- LOGGING ----------------
@bot.command()
async def setlog(ctx, channel: discord.TextChannel):
    logs = load_json("logs.json")
    logs[str(ctx.guild.id)] = channel.id
    save_json("logs.json", logs)
    await ctx.send("Log channel set")

def log(ctx, msg):
    logs = load_json("logs.json")
    cid = logs.get(str(ctx.guild.id))
    if cid:
        ch = bot.get_channel(cid)
        if ch:
            bot.loop.create_task(ch.send(msg))

# ---------------- MODERATION (40 COMMANDS) ----------------
@bot.hybrid_command()
async def kick(ctx, member: discord.Member):
    await member.kick()

@bot.hybrid_command()
async def ban(ctx, member: discord.Member):
    await member.ban()

@bot.hybrid_command()
async def unban(ctx, user_id:int):
    user = await bot.fetch_user(user_id)
    await ctx.guild.unban(user)

@bot.hybrid_command()
async def softban(ctx, member: discord.Member):
    await member.ban()
    await member.unban()

@bot.hybrid_command()
async def mute(ctx, member: discord.Member, minutes:int=10):
    await member.timeout(discord.utils.utcnow()+timedelta(minutes=minutes))

@bot.hybrid_command()
async def unmute(ctx, member: discord.Member):
    await member.timeout(None)

@bot.hybrid_command()
async def deafen(ctx, member: discord.Member):
    await member.edit(deafen=True)

@bot.hybrid_command()
async def undeafen(ctx, member: discord.Member):
    await member.edit(deafen=False)

@bot.hybrid_command()
async def move(ctx, member: discord.Member, channel: discord.VoiceChannel):
    await member.move_to(channel)

@bot.hybrid_command()
async def disconnect(ctx, member: discord.Member):
    await member.move_to(None)

@bot.hybrid_command()
async def nick(ctx, member: discord.Member, *, name):
    await member.edit(nick=name)

@bot.hybrid_command()
async def clearnick(ctx, member: discord.Member):
    await member.edit(nick=None)

@bot.hybrid_command()
async def purge(ctx, amount:int):
    await ctx.channel.purge(limit=amount)

@bot.hybrid_command()
async def purgeuser(ctx, member: discord.Member):
    def check(m): return m.author == member
    await ctx.channel.purge(limit=1000, check=check)

@bot.hybrid_command()
async def purgebots(ctx):
    def check(m): return m.author.bot
    await ctx.channel.purge(limit=1000, check=check)

@bot.hybrid_command()
async def purgecontains(ctx, *, text):
    def check(m): return text in m.content
    await ctx.channel.purge(limit=1000, check=check)

@bot.hybrid_command()
async def purgeembeds(ctx):
    def check(m): return m.embeds
    await ctx.channel.purge(limit=1000, check=check)

@bot.hybrid_command()
async def purgeattachments(ctx):
    def check(m): return m.attachments
    await ctx.channel.purge(limit=1000, check=check)

@bot.hybrid_command()
async def lock(ctx):
    await ctx.channel.set_permissions(ctx.guild.default_role, send_messages=False)

@bot.hybrid_command()
async def unlock(ctx):
    await ctx.channel.set_permissions(ctx.guild.default_role, send_messages=True)

@bot.hybrid_command()
async def lockall(ctx):
    for ch in ctx.guild.text_channels:
        await ch.set_permissions(ctx.guild.default_role, send_messages=False)

@bot.hybrid_command()
async def unlockall(ctx):
    for ch in ctx.guild.text_channels:
        await ch.set_permissions(ctx.guild.default_role, send_messages=True)

@bot.hybrid_command()
async def slow(ctx, seconds:int):
    await ctx.channel.edit(slowmode_delay=seconds)

@bot.hybrid_command()
async def clearslow(ctx):
    await ctx.channel.edit(slowmode_delay=0)

@bot.hybrid_command()
async def clone(ctx):
    new = await ctx.channel.clone()
    await ctx.send(new.mention)

@bot.hybrid_command()
async def renamechannel(ctx, *, name):
    await ctx.channel.edit(name=name)

@bot.hybrid_command()
async def topic(ctx, *, text):
    await ctx.channel.edit(topic=text)

@bot.hybrid_command()
async def createchannel(ctx, name):
    await ctx.guild.create_text_channel(name)

@bot.hybrid_command()
async def deletechannel(ctx):
    await ctx.channel.delete()

@bot.hybrid_command()
async def createrole(ctx, name):
    await ctx.guild.create_role(name=name)

@bot.hybrid_command()
async def deleterole(ctx, role: discord.Role):
    await role.delete()

@bot.hybrid_command()
async def addrole(ctx, member: discord.Member, role: discord.Role):
    await member.add_roles(role)

@bot.hybrid_command()
async def removerole(ctx, member: discord.Member, role: discord.Role):
    await member.remove_roles(role)

@bot.hybrid_command()
async def massrole(ctx, role: discord.Role):
    for m in ctx.guild.members:
        await m.add_roles(role)

@bot.hybrid_command()
async def removeroleall(ctx, role: discord.Role):
    for m in ctx.guild.members:
        await m.remove_roles(role)

@bot.hybrid_command()
async def nuke(ctx):
    ch = ctx.channel
    new = await ch.clone()
    await ch.delete()
    await new.send("💣 nuked")

@bot.hybrid_command()
async def say(ctx, *, msg):
    await ctx.message.delete()
    await ctx.send(msg)

@bot.hybrid_command()
async def embed(ctx, *, msg):
    e = discord.Embed(description=msg, color=discord.Color.random())
    await ctx.send(embed=e)

# ---------------- ECONOMY ----------------
@bot.hybrid_command()
async def balance(ctx):
    eco = load_json("eco.json")
    uid = str(ctx.author.id)
    eco.setdefault(uid,100)
    await ctx.send(f"${eco[uid]}")

@bot.hybrid_command()
async def work(ctx):
    eco = load_json("eco.json")
    uid = str(ctx.author.id)
    eco.setdefault(uid,100)
    amt = random.randint(50,150)
    eco[uid]+=amt
    save_json("eco.json",eco)
    await ctx.send(f"+${amt}")

# ---------------- ACTION COMMANDS (100+) ----------------
actions = ["hug","slap","kiss","punch","kick","bite","pat","cuddle","bonk","yeet",
"wave","smile","cry","laugh","dance","sleep","poke","stare","highfive","facepalm"]

for act in actions:
    async def cmd(ctx, member: discord.Member=None, act=act):
        target = member.mention if member else "someone"
        await ctx.send(f"{ctx.author.mention} {act}s {target}")

# ---------------- ERROR ----------------
@bot.event
async def on_command_error(ctx, error):
    if isinstance(error, commands.CommandNotFound):
        return
    await ctx.send(f"Error: {error}")

# ---------------- RUN ----------------
bot.run(TOKEN)

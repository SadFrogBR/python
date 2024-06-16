import discord
from discord.ext import commands, tasks
import facebook
import aiohttp
import io

# Configurações do bot
TOKEN = 'token do bot do discord'
FACEBOOK_ACCESS_TOKEN = 'token do facebook'
PROFILE_ID = 'id perfil'
CHANNEL_ID = 'id canal'


# Inicializar clientes
intents = discord.Intents.default()
intents.messages = True
intents.guilds = True
client = commands.Bot(command_prefix='!', intents=intents)
fb_graph = facebook.GraphAPI(access_token=FACEBOOK_ACCESS_TOKEN)

# Função para buscar postagens do Facebook
def get_facebook_posts():
    try:
        posts = fb_graph.get_connections(id=PROFILE_ID, connection_name='posts')
        if 'data' in posts:
            print("Postagens encontradas:", posts['data'])
            return posts['data']
        else:
            print("Nenhuma postagem encontrada.")
            return []
    except facebook.GraphAPIError as e:
        print(f"Erro ao buscar postagens do Facebook: {e}")
        return []

# Conjunto para armazenar IDs das postagens já enviadas
post_ids_sent = set()

# Função para enviar postagens ao Discord
async def send_posts_to_discord():
    channel = client.get_channel(CHANNEL_ID)
    posts = get_facebook_posts()
    for post in posts:
        post_id = post['id']
        # Verificar se a postagem já foi enviada
        if post_id not in post_ids_sent:
            # Adicionar o ID da postagem ao conjunto
            post_ids_sent.add(post_id)
            # Construir o link da postagem
            post_url = f"https://www.facebook.com/{PROFILE_ID}/posts/{post_id}"
            # Mensagem personalizada
            message = f"Nova postagem na página! Confira: {post_url}"
            # Pegar os primeiros 100 caracteres da mensagem
            post_message = post.get('message', '')[:100]
            # Enviar a mensagem embutida com o link
            embed = discord.Embed(description=f"{message}\n{post_message}")
            # Adicionar uma imagem à mensagem embutida (opcional)
            embed.set_image(url='https://ddragon.leagueoflegends.com/cdn/img/champion/splash/Ashe_9.jpg')
            await channel.send(embed=embed)

# Tarefa agendada para buscar e enviar postagens regularmente
@tasks.loop(seconds=10)
async def fetch_and_send_posts():
    await send_posts_to_discord()

# Evento de inicialização do bot
@client.event
async def on_ready():
    print(f'{client.user.name} has connected to Discord!')
    fetch_and_send_posts.start()

# Comando para iniciar manualmente o envio de postagens (opcional)
@client.command()
async def getposts(ctx):
    await send_posts_to_discord()
    await ctx.send('Posts from Facebook fetched and sent to Discord!')

# Executar o bot
client.run(TOKEN)

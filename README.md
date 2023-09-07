from nonebot import on_command
from nonebot.adapters.onebot.v11 import MessageEvent,GroupMessageEvent, Message
from nonebot.params import RegexGroup,CommandArg
import httpx
import re
from bs4 import BeautifulSoup

__zx_plugin_name__ = "tmp查询"
__plugin_usage__ = """
usage：
    p
    指令：
        tmp查询[id]
""".strip()
__plugin_des__ = "l"
__plugin_cmd__ = ["[城市]天气/天气[城市]"]
__plugin_type__ = ("一些工具",)
__plugin_version__ = 0.1
__plugin_author__ = "wenwen12305"
__plugin_settings__ = {
    "level": 5,
    "default_status": True,
    "limit_superuser": False,
    "cmd": ["tmp查询"],
}


tmp_id = on_command("查询", priority=5, block=True)
tmp_sever=on_command("服务器", priority=5, block=True)
tmp_road=on_command("路况", priority=5, block=True)

def search_city(html,id):
    exp=BeautifulSoup(html,'html.parser')
    cars=exp.find('span',id='traffic_players_'+str(id)).contents
    status=exp.find('span',id='traffic_status_'+str(id)).contents
    if status==['Moderate']:
        status='正常'
    elif status==['Low'] or status==['Empty']:
        status='通畅'
    elif status==['Congested']:
        status='爆满'
    elif status==['Heavy']:
        status='拥堵'
    return ' '.join(['路况:',status,'| 玩家:']+cars)

def if_ban(a:bool,b:str):
        if a:
            return ("是\n解禁时间:"+str(b))
        return "否"

def null_check(str:str):
    if str:
        return str
    return "无"

def Detail(server,check_id,detail,output_id):
    if server['id']==check_id:
        detail[output_id]=[server['players'],server['queue'],server['maxplayers']]

def to_string_if_on(a:bool):
    if a:
        return '在线'
    return '离线'

lot_of_parrel='='*19


@tmp_id.handle()
async def handle_function(event: MessageEvent,args: Message = CommandArg()):
    id=args.extract_plain_text()
    if not(str(id).isdigit()):
        await tmp_id.finish("id输入错误！")
    url="https://api.truckersmp.com/v2/player/"+str(id)
    
    async with httpx.AsyncClient(verify=False, timeout=60) as client:
            response = await client.get(url)
    if response.status_code==404 or response.json()['error']:
        await tmp_id.finish("查询失败，请重新输入")
    ans=["查询完成\nTMPID:",str(response.json()['response']['id']),"\n玩家名称:",response.json()['response']['name'],'\n玩家类别:',response.json()['response']['groupName'],'\n注册时间:',response.json()['response']['joinDate'],"\n是否封禁:",if_ban(response.json()['response']['banned'],response.json()['response']['bannedUntil']),'\n年内封禁:',str(response.json()['response']['bansCount']),'\nTMP链接:https://truckersmp.com/user/',str(id),'\nSteam链接:https://steamcommunity.com/profiles/',response.json()['response']['steamID'],"\n所属车队:",null_check(response.json()['response']['vtc']['name'])]
    await tmp_id.finish(''.join(ans))

@tmp_road.handle()
async def handle_function(event: MessageEvent,args: Message = CommandArg()):
    id=args.extract_plain_text()
    if re.match('s1(服)?',str(id),re.I):
        async with httpx.AsyncClient(verify=False, timeout=60) as client:
            response = await client.get('https://traffic.krashnz.com/ets2/sim1')
        response=response.text
        ans=['地区:加来-杜伊斯堡道路\n',search_city(response,134),'\n',lot_of_parrel,'\n地区:加来\n',search_city(response,55),'\n',lot_of_parrel,'\n地区:加来（十字路口）\n',search_city(response,217),'\n',lot_of_parrel,'\n地区:杜伊斯堡\n',search_city(response,14),'\n',lot_of_parrel,'\n地区:杜塞尔多夫\n',search_city(response,15),'\n',lot_of_parrel,'\n地区:里尔\n',search_city(response,57),'\n',lot_of_parrel,'\n地区:阿尔卑斯高山公路(道路)\n',search_city(response,1016)]
        await tmp_road.finish(''.join(ans))
    if re.match('s2(服)?',str(id),re.I):
        async with httpx.AsyncClient(verify=False, timeout=60) as client:
            response = await client.get('https://traffic.krashnz.com/ets2/sim2')
        response=response.text
        ans=['地区:加来-杜伊斯堡道路\n',search_city(response,134),'\n',lot_of_parrel,'\n地区:加来\n',search_city(response,55),'\n',lot_of_parrel,'\n地区:杜伊斯堡\n',search_city(response,14),'\n',lot_of_parrel,'\n地区:杜塞尔多夫\n',search_city(response,15),'\n',lot_of_parrel,'\n地区:阿尔卑斯高山公路(道路)\n',search_city(response,1016)]
        await tmp_road.finish(''.join(ans))
    if re.match('p(服)?',str(id),re.I):
        async with httpx.AsyncClient(verify=False, timeout=60) as client:
            response = await client.get('https://traffic.krashnz.com/promods/pm')
        response=response.text
        ans=['地区:希尔科内斯城市\n',search_city(response,615),'\n',lot_of_parrel,'\n地区:希尔科内斯-采石场\n',search_city(response,793),'\n',lot_of_parrel,'\n地区:阿克雷里（冰岛）\n',search_city(response,389),'\n',lot_of_parrel,'\n地区:维克（冰岛）\n',search_city(response,391),'\n',lot_of_parrel,'\n地区:北峡湾（冰岛）\n',search_city(response,393),'\n',lot_of_parrel,'\n地区:朗伊尔城隧道（迷宫）\n',search_city(response,794)]
        await tmp_road.finish(''.join(ans))
    tmp_road.finish('输入无效')

@tmp_sever.handle()
async def handle_function(event: MessageEvent,args: Message = CommandArg()):
    async with httpx.AsyncClient(verify=False, timeout=60) as client:
        response = await client.get('https://api.truckersmp.com/v2/servers')
    detail=[[],[],[],[],[]]
    server_status=[[],[],[],[],[]]
    server_max_player=[0,0,0,0,0]
    server_player=[0,0,0,0,0]
    total=0
    for server in response.json()['response']:
        if server['id']==4:
            server_player[0]=server['players']
            server_max_player[0]=server['maxplayers']
            server_status[0]=1
        if server['id']==41:
            server_player[1]=server['players']
            server_max_player[1]=server['maxplayers']
            server_status[1]=1
        if server['id']==8:
            server_player[2]=server['players']
            server_max_player[2]=server['maxplayers']
            server_status[2]=1
        if server['id']==31:
            server_player[3]=server['players']
            server_max_player[3]=server['maxplayers']
            server_status[3]=1
        if server['id']==35:
            server_player[4]=server['players']
            server_max_player[4]=server['maxplayers']
            server_status[4]=1
        total+=server['players']
    ans='在线玩家:'+str(total)+'\n'
    servers=['Simulation1','Simulation2','Arcade','Promods','[SGP] Simulation']
    for i in range(5):
        ans+=lot_of_parrel+'\n地区:'+servers[i]+'\n状态:'+to_string_if_on(server_status[i])+'\n人数:'+str(server_player[i])+'/'+str(server_max_player[i])+'\n'
    await tmp_sever.finish(ans)

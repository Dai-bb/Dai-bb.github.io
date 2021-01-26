# Dai-bb.github.io
import urllib.request
import urllib.parse
import re
import xlwt
from bs4 import  BeautifulSoup

def main():
    #爬取网页
    beasurl='https://movie.douban.com/top250?start='
    datalist=getData(beasurl)
    #保存数据
    savepath='.\\豆瓣电影TOP250.xls'
    saveData(datalist,savepath)

#网页链接的规则
findLink= re.compile(r'<a href="(.*?)">') #创建正则表达式,表示规则（字符串模式）
#图片规则
findImg=re.compile(r'<img.*src="(.*?)"',re.S)
#影片的片名
findTitle=re.compile(r'<span class="title">(.*)</span>')
#电影评分
findRating=re.compile(r'<span class="rating_num" property="v:average">(.*)</span>')
#电影评价人数
findJudge=re.compile(r'<span>(\d*)人评价</span>')
#概况
findInq=re.compile(r'<span class="inq">(.*)</span>')
#影片的相关内容
findBd=re.compile(r'<p class="">(.*?)</p>',re.S)


#爬取网页
def getData(beasurl):
    datalist = []
    for i in range(0,10):
        url=beasurl+str(i*25)
        html=askURL(url) #注意要返回值的保存
        #解析网页
        soup=BeautifulSoup(html,'html.parser')
        for item in soup.find_all('div',class_="item"):
            data=[]
            item=str(item)


            link=re.findall(findLink,item)[0]
            data.append(link)

            img=re.findall(findImg,item)
            data.append(img)

            title=re.findall(findTitle,item)[0]
            if len(title)==2:
                ctitle=title[0]
                data.append(ctitle)
                otitle=title[1]
                data.append(otitle)
            else:
                data.append(title[0])
                data.append('')
            rating=re.findall(findRating,item)[0]
            data.append(rating)
            judgeNum=re.findall(findJudge,item)[0]
            data.append(judgeNum)
            inq=re.findall(findInq,item)
            if len(inq)!=0:
                inq=inq[0].replace('。','')
                data.append(inq)
            else:
                data.append('')

            bd=re.findall(findBd,item)[0]
            bd=re.sub('<br(\s+)?/>(\s+)?', '', bd)
            bd = re.sub('/', '', bd)
            data.append(findBd)
            datalist.append(data)

    return datalist

#得到指定ulr的网页内容
def askURL(url):
    head={
        'User-Agent':'Mozilla/5.0(Windows NT 10.0;WOW64) AppleWebKit / 537.36(KHTML, likeGecko) Chrome / 84.0.4147.135Safari / 537.36'
    }
    rep=urllib.request.Request(url=url,headers=head)
    html=''
    try:
        response=urllib.request.urlopen(rep)
        html=response.read().decode('utf-8')
    except urllib.error.URLerror as e:
        if hasattr(e,'code'):
            print(e.code)
        if hasattr(e,'reason'):
            print(e.reason)
    return html #注意要返回值

def saveData(datalist,savepath):       
    print('save..........')
    book=xlwt.Workbook(encoding='utf-8',style_compression=0)
    sheet=book.add_sheet('豆瓣电影TOP250',cell_overwrite_ok=True)
    col=('网页链接','图片链接','片名','外国名','评分','评分人数','概况','相关信息')
    for i in range(0,8):
        sheet.write(0,i,col[i]) #列名
    for i in range(0,250):
        print('第%d条' %(i+1))
        datalist
        for j in range(0,8):
             sheet.write(i+1,j,data[j])
    book.save(savepath)

if __name__ == '__main__':
    main()
    print('爬取完毕')

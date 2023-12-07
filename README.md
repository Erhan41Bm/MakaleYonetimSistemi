# MakaleYonetimSistemi
import mysql.connector
from datetime import datetime

mydb = mysql.connector.connect(host= 'db4free.net',user = "makaleyonetim",password = "12345678",database = "makaleyonetimdb",port = 3306)
cursor = mydb.cursor()
def acılısekranı():
     print("\n1 - Giriş Yap \n2 - Üye ol\n3 - Çıkış Yap")
     secim = input("Seciminiz:")
     if secim=="1":
        giris()
     elif secim=="2":
        kayitol()
     elif secim=="3":
        print("Çıkış yapılıyor. Hoşça kalın!")
        exit()
     else:
         print("Hatalı bir seçim yaptınız...")
         acılısekranı()

def kayitol():
    kullaniciadi = input("kulanici adınız:")
    kullanici_query = "SELECT * FROM Kullanici WHERE adi=%s"
    cursor.execute(kullanici_query, (kullaniciadi,))
    kullanici = cursor.fetchall()
    if (len(kullanici) > 0):
        print("Bu kullanici adi daha önce alınmış.Lütfen başka kullanıcı adı ile kayıt olunuz...")
        kayitol()
    sifre = input("sifreniz:")
    tipi = input("kullanıcı rolunuz(Yazar için 1;Editör için 2;Hakem için 3) Seçiminiz:")
    if (tipi == "1"):
        tip = "Yazar"
    elif (tipi == "2"):
        tip = "Editör"
    elif (tipi == "3"):
        tip = "Hakem"
    else:
        print("Kullanıcı tipi için lütfen doğru değer seçiniz.")
        kayitol()

    try:
        query = "INSERT INTO Kullanici (adi, sifre, tipi) VALUES (%s, %s, %s)"
        values = (kullaniciadi,sifre,tip)
        cursor.execute(query, values)
        mydb.commit()
        print("Başarılı bir şekilde sisteme kayıt oldunuz.Lütfen Sisteme giriş yapınız.")
        acılısekranı()
    except mysql.connector.Error as err:
        print("Hata:", err)

def giris():
    try:
        kullaniciadi = input("kulanici adınız:")
        sifre = input("sifreniz:")
        query = "SELECT * FROM Kullanici WHERE adi = %s AND sifre = %s"
        values = (kullaniciadi, sifre)
        cursor.execute(query, values)
        user = cursor.fetchone()
        if user:
            print("Giriş başarılı. Hoş geldiniz,", user[1])
            if  user[3]=="Yazar":
                print("YAZAR MENUSU\n---------------")
                yazarmenusu(user[0])
            elif user[3]=="Editör":
                print("EDİTÖR MENUSU\n---------------")
                editormenusu(user[0])
            elif user[3]=="Hakem":
                print("HAKEM MENUSU\n---------------")
                hakemmenusu(user[0])
        else:
            print("Kullanıcı adı veya şifre hatalı.Lütfen tekrar kontrol ediniz.")
            giris()
    except mysql.connector.Error as err:
        print("Hata:", err)
        return None
def MakaleEkle(basligi,yazarlari,yukleyen_eposta,yukleyen_id,yukleyen_kurum ):

    sql="INSERT INTO Makale(basligi,yazarlari,yukleyen_eposta, durumu,yukleyen_id,editor_id,hakem_id,yukleyen_kurum,yuklenme_tarih) VALUES (%s,%s,%s,%s,%s, %s, %s,%s,%s)"
    values=(basligi,yazarlari,yukleyen_eposta,"Yüklendi",yukleyen_id,0,0,yukleyen_kurum,datetime.now())
    cursor.execute(sql,values)
    try:
        mydb.commit()
    except mysql.connector.Error as err:
        print('Hata:',err)
    finally:
        print('Makale sisteme eklendi ve durumu Yüklendi olarak ayarlandı\n--------------------')


def YazarMakaleListele(id):
    query = "SELECT * FROM Makale WHERE yukleyen_id = %s"
    cursor.execute(query, (id,))
    makaleler =cursor.fetchall()
    if makaleler:
        print( "-------------------------\nSistemde yüklü olan", len(makaleler)," adet makaleniz vardır.\n-------------------")
        for makale in makaleler:
            print("Makale ID:", makale[0])
            print("Başlık:", makale[1])
            print("Yazarlar:", makale[2])
            print("Durumu:", makale[4])
            print("Yükleme Tarihi:", makale[9])
            print("--------------------------\n")
    else:
        print("-----------------------\nHenüz sisteme yüklenmiş makaleniz bulunmamaktadır.\n------------------")
def TumMakaleListele():
    query = "SELECT * FROM Makale "
    cursor.execute(query, ())
    makaleler = cursor.fetchall()
    if makaleler:
        print( "Sistemde toplam", len(makaleler),"adet makale vardır:")
        print("-------------------------")
        for makale in makaleler:
            query1 = "SELECT * FROM Kullanici WHERE id = %s"
            cursor.execute(query1, (makale[5],))
            yazar=cursor.fetchone()
            print("Makale ID:", makale[0])
            print("Başlık:", makale[1])
            print("Yazarlar:", makale[2])
            print("Durumu:", makale[4])
            print("Makaleyi Yükleyen:",yazar[1])
            print("Makaleye atanan yazar:", makale[4])
            print("Yükleme Tarihi:", makale[9])
            print("--------------------------\n")
    else:
        print("Henüz sisteme yüklenmiş makale bulunmamaktadır.")

def HakemsizMakaleListele():
    query = "SELECT * FROM Makale WHERE hakem_id=%s "
    cursor.execute(query, (0,))
    makaleler = cursor.fetchall()
    if makaleler:
        print( "Henüz hakem atamaması yapılmamış",len(makaleler),"adet makale vardır.")
        print("-------------------------")
        for makale in makaleler:
            query1 = "SELECT * FROM Kullanici WHERE id = %s"
            cursor.execute(query1, (makale[5],))
            yazar=cursor.fetchone()
            print("Makale ID:", makale[0])
            print("Başlık:", makale[1])
            print("Yazarlar:", makale[2])
            print("Durumu:", makale[4])
            print("Makaleyi Yükleyen:",yazar[1])
            print("Makaleye atanan yazar:", makale[4])
            print("Yükleme Tarihi:", makale[9])
            print("--------------------------\n")
    else:
        print("Sistemde hakem ataması yapılmamış makale bulunmamaktadır. \n-----------------------")

def HakemListele():
    query = "SELECT * FROM Kullanici WHERE tipi=%s "
    cursor.execute(query, ("Hakem",))
    hakemler = cursor.fetchall()
    if hakemler:
        print( "Sistemde kayıtlı olan Hakemler:")
        print("-------------------------")
        for hakem in hakemler:
            print("Hakem Kullanıcı ID:", hakem[0])
            print("Hakem Kullanıcı Adı:", hakem[1])
            print("--------------------------\n")
    else:
        print("Henüz sisteme kayıtlı hakem bulunmamaktadır.")

def HakemAta(makale_id,hakem_id,editor_id):
    update_query = "UPDATE Makale SET durumu =%s, hakem_id= %s, editor_id=%s WHERE id = %s"
    cursor.execute(update_query, ("Değerlendirmede",hakem_id,editor_id,makale_id,))
    mydb.commit()
    makale_query = "SELECT * FROM Makale WHERE id=%s "
    cursor.execute(makale_query ,(makale_id,))
    makale=cursor.fetchone()
    hakem_query = "SELECT * FROM Kullanici WHERE id=%s "
    cursor.execute(hakem_query, (hakem_id,))
    hakem = cursor.fetchone()
    print("-----------------\n",makale[1]," başlıklı makaleye",hakem[1],
          "kullanıcı adlı hakem atandı ve makale durumu Değerelendirmede olarak güncellendi."
          "\n------------------------")
def HakemMakaleListele(id):
    query = "SELECT * FROM Makale WHERE hakem_id = %s"
    cursor.execute(query, (id,))
    makaleler = cursor.fetchall()
    if makaleler:
        print("-------------------------\nAdınıza zimmetlenmiş", len(makaleler),
              " adet makale vardır.\n-------------------")
        for makale in makaleler:
            print("Makale ID:", makale[0])
            print("Başlık:", makale[1])
            print("Yazarlar:", makale[2])
            print("Durumu:", makale[4])
            print("Yükleme Tarihi:", makale[9])
            print("--------------------------\n")
    else:
        print("-----------------------\nHenüz adınıza zimmetlenmiş makale bulunmamaktadır.\n------------------")

def MakaleDurumuGuncelle():
    makale_id = int(input("Durumunu güncellemek istediğiniz makalenin id sini giriniz:"))
    durum = input("Seçtiğiniz makalenin durumunu nasıl güncellemek istersiniz?(Kabul:1,Red:2,Değerlendirmede:3)")
    if durum == "1":
        makale_durumu = "Kabul Edildi"
    elif durum == "2":
        makale_durumu = "Ret Edildi"
    elif durum == "3":
        makale_durumu = "Değerlendirmede"
    else:
        print("makale durumu için yanlış seçim yaptınız...")
        MakaleDurumuGuncelle()
    update_query = "UPDATE Makale SET durumu =%s WHERE id = %s"
    cursor.execute(update_query, (makale_durumu, makale_id,))
    mydb.commit()
    makale_query = "SELECT * FROM Makale WHERE id=%s "
    cursor.execute(makale_query, (makale_id,))
    makale = cursor.fetchone()
    print("-----------------\n", makale[1], " başlıklı makalenin durumu", makale_durumu, "olarak güncellendi."
          "\n------------------------")

def yazarmenusu(id):
    print("Mevcut makalelerinizi listelemek için:1\nYeni Makale eklemek için:2\nÇıkış yapmak için:3")
    secim=input("Secim Yapınız:")
    if secim=="1":
        YazarMakaleListele(id)
    elif secim=="2":
        baslik = input("Makalenizin başlığı:")
        yazarlar = input("Makalenizin yazarlari(yazarları virgülle ayırınız.):")
        eposta = input("E-posta adresiniz:")
        kurum = input("Çalıştığınız Kurum:")
        MakaleEkle(baslik,yazarlar,eposta,id,kurum)
    elif secim=="3":
        acılısekranı()
    else:
        print("Hatalı seçim yaptınız...")

    yazarmenusu(id)


def editormenusu(id):
    print("Sistemde yüklü makaleleri listelemek için:1\nHakem ataması yapılmamış makaleleri listelemek "
          "için:2\nBir makaleye Hakem ataması yapmak için:3\nÇıkış yapmak için:4")
    secim = input("Secim Yapınız:")
    if secim == "1":
        TumMakaleListele()

    elif secim=="2":
        HakemsizMakaleListele()

    elif secim=="3":
        makale_id = int(input("Hakem atamak istediğiniz makalenin id sini giriniz:"))
        HakemListele()
        hakem_id = int(input("Seçtiğiniz makaleye atama yapmak istediğiniz hakemin idsini seçiniz:"))
        HakemAta(makale_id,hakem_id,id)

    elif secim=="4":
        acılısekranı()
    else:
        print("Hatalı seçim yaptınız...")
    editormenusu(id)
def hakemmenusu(id):
    print("Adınıza zimmetlenmiş makaleleri listelemek için:1\nBir makalenin durumunu güncellemek için:2\nÇıkış yapmak için:3")
    secim = input("Secim Yapınız:")
    if secim == "1":
        HakemMakaleListele(id)
    elif secim=="2":
        MakaleDurumuGuncelle()
    elif secim=="3":
        acılısekranı()
    else:
        print("Hatalı seçim yaptınız...")
    hakemmenusu(id)


print("Makale yönetim Sistemine hoşgeldiniz.")
acılısekranı()

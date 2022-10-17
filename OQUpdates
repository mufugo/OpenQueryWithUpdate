/*

Bu sorgu ile, OPENQUERY kullanarak başka sunucuda bulunan verileri, bizim sunucumuzda olmayan veriler ile iç içe OPENQUERY yaparak kontrol ettirdik.
Ayrıca çıkan sonucu da #data adı altında bir temp tablo oluşturarak buraya INSERT attık.

Ardından temp olarak oluşturduğumuz tabloda duran rowcounta göre WHILE döngümüzü çalıştırdık ve eşleşen kayıtları
while döngüsü içinde update ettik. Update sonrası Counter 1 arttırdığımız için sayaç, toplam satır sayısına eşitlenince
while döngüsü sonlandı ve ardından temp tabloyu sildi.

Başlangıçta da eğer temp tablo varsa sildirdik.

*/

DECLARE @Counter INT , @Count INT
SELECT @Counter = 1 , @Count = COUNT(CashHeaderID) from dbo.trCashHeader WHERE CashCurrAccCode IS NULL
DECLARE @updatedata NVARCHAR(1000)
DECLARE @UUID NVARCHAR(1000)


-- data adında bir tempdb var mı kontrol ettik. Var ise sildirdik.
IF OBJECT_ID('tempdb..#data') IS NOT NULL 
BEGIN 
    DROP TABLE #data
END


-- data adında geçici tablomuzu oluşturduk.
CREATE TABLE #data 
(
CashCurrAccCode nvarchar(max),
CashHeaderID uniqueidentifier,
RowNumber bigint
)

-- dosya adı her satırda farklı olması ve tek bir satır döndürmesi için işlem yapıldı geçici bir tablo oluşturmak gibi bir işlem
;WITH data AS  
(  
	-- bu select tablodaki tüm satırları ORDER BY file_name asc olacak şekilde ayarladı ve onlara sıra numarası atadı
    
    SELECT CashCurrAccCode , CashHeaderID , ROW_NUMBER() OVER (ORDER BY CashHeaderID) AS RowNumber FROM OPENQUERY ([MAINSERVER], '
		SELECT 
		CashCurrAccCode,CashHeaderID        FROM  MAINDATA.dbo.trCashHeader
		        
		        
		WHERE CashHeaderID IN (
		SELECT * FROM OPENQUERY ([SECONDSERVER],
		''SELECT CashHeaderID FROM SECONDDATA.dbo.trCashHeader WHERE CashCurrAccCode IS NULL  '')

		) AND CompanyCode=''2'' AND DocumentDate BETWEEN ''20220701'' AND ''20221231'' ')
		    
)
-- yukarıda with olarak verdiğimiz data içinde dönen verileri, geçici tablomuz olan #data içine kayıt ettik.
 	INSERT INTO #data 
 	SELECT * FROM data  

-- while döngümüzü başlattık ve kayıt sayımıza eşit veya daha altında olana kadar devam ettirdik.
WHILE(@Counter <= @Count)
BEGIN

-- updatedata ve UUID kayıtları, geçici tablomuzda yer alan bilgilere göre SET edildi, burada koşul olarak kaçıncı satırda(sayaç) isek o değerler getirildi.

SELECT @updatedata = CashCurrAccCode, @UUID = CashHeaderID FROM #data WHERE RowNumber = @Counter

-- Update sorgumuz
	UPDATE  dbo.trCashHeader
	SET     CashCurrAccCode = @updatedata
	WHERE   CashHeaderID = @UUID
-- Update başarılı olduktan sonra sayacımızı 1 arttırdık
		SELECT @Counter += 1

END
-- while döngüsü sonlandıktan sonra temp tabloyu sildik.
DROP TABLE #data
GO

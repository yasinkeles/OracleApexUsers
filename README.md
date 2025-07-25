#### Oracle Apex'te son kullanıcı yetkilendirmeleri için bir tablo oluşturuyoruz .(USERS)

![image](https://github.com/user-attachments/assets/d71ca16e-4eef-4091-b2fd-4f5c8628698b)


* USER_ID	NUMBER
* USERNAME	VARCHAR2
* PASSWORD	VARCHAR2
* DEPARTMENT	VARCHAR2


#### Yetkilendirmeleri kontrol edecek bir fonksiyon oluşturuyoruz .(CHECK_USER)

![image](https://github.com/user-attachments/assets/d22bb828-a687-4e6c-b8cf-8dbb4d2a598f)
```
create or replace FUNCTION           CHECK_USER(
    p_username IN VARCHAR2,
    p_password IN VARCHAR2
) RETURN BOOLEAN AS
    l_count NUMBER := 0;
BEGIN
    -- Debug log ekleyelim
    --DBMS_OUTPUT.PUT_LINE('Kontrol edilen kullanıcı: ' || p_username);
    
    SELECT COUNT(*)
    INTO l_count
    FROM DASHBOARD.USERS
    WHERE USERNAME = p_username
      AND PASSWORD = p_password;

    IF APEX_UTIL.IS_LOGIN_PASSWORD_VALID (
        p_username => p_username,
        p_password => p_password
        ) THEN
        return true;
    END IF;

    IF l_count > 0 THEN
        --DBMS_OUTPUT.PUT_LINE('Kullanıcı bulundu: ' || p_username);
        RETURN TRUE;
    ELSE
        --DBMS_OUTPUT.PUT_LINE('Kullanıcı bulunamadı: ' || p_username);
        null;
    END IF;

    RETURN FALSE;
END;
/
```

#### Yetkilendirmelerin geçerli olacağı uygulamada bir Authorization Schemes oluşturuyoruz .

![image](https://github.com/user-attachments/assets/9605a969-33b4-4d2b-8a37-8d13ce2c2992)

#### Her grup için ayrı yetkilendirmeler kullanıyoruz, şema tipini "Exist SQL Query" olarak seçiyoruz.
![image](https://github.com/user-attachments/assets/4c8c36ea-fecb-4692-ae12-51821aaf8656)

```
Admin kullanıcılar için 

SELECT 1 
FROM APEX_WORKSPACE_APEX_USERS 
WHERE USER_NAME = :APP_USER 
AND IS_ADMIN = 'Yes';
```
```
Örn: Muhasebe departmanı için

SELECT 1 
FROM DASHBOARD.USERS 
WHERE USERNAME = :APP_USER 
AND DEPARTMENT = 'MUHASEBE'

UNION ALL

SELECT 1 
FROM APEX_WORKSPACE_APEX_USERS 
WHERE USER_NAME = :APP_USER 
AND IS_ADMIN = 'Yes';
```
```
Örn: Muhasebe ve Lojistik için

SELECT 1 
FROM DASHBOARD.USERS 
WHERE UPPER(TRIM(USERNAME)) = UPPER(TRIM(:APP_USER)) 
AND DEPARTMENT IN ('LOJISTIK', 'MUHASEBE')

UNION ALL

SELECT 1 
FROM APEX_WORKSPACE_APEX_USERS 
WHERE UPPER(TRIM(USER_NAME)) = UPPER(TRIM(:APP_USER)) 
AND IS_ADMIN = 'Yes';
```


#### Departman bazlı yetkilendirmeyi kullanmak için "Navigation Menu" altından bir menü oluşturup her sayfanın "Autthorization Scheme" seçeneğini ilgili departan için oluşturulan şemayı seçiyoruz

![image](https://github.com/user-attachments/assets/8ff6f368-6293-4c5b-b30c-6ef22d48b8d6)
![image](https://github.com/user-attachments/assets/04f9fc17-aa07-4b43-9673-d4e39834f47e)

#### son olarak bir Autentication şema oluşturup sql imizi gösterip varsayılan yapıyoruz

<img width="1720" height="1245" alt="Edit-Authentication-Scheme-07-25-2025_02_46_PM" src="https://github.com/user-attachments/assets/6784e282-28ca-41c3-af52-f357cd1e9bab" />


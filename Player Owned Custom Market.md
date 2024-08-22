# How TO MAKE YOUR OWN MARKET
----------------------------------------------------------------
## Creating the Market

#### Getting the Market assets.

> [!NOTE]
>**DO THIS ALL FROM BO  ( BACK OFFICE )**

Go to PLAYERS and select **"admin"**   <sup>Or what ever your player name is.</sup>

Go to Inventory Tab and Add the following Items:
1. Static Core ANY SIZE 
2. MARKET POD
3. MARKET UNIT M
   >[!IMPORTANT]
   >Market Unit M **SUPER IMPORTAND** NOT THE OTHER ONES !!!!! ONLY M !!!!!

> [!TIP]
>The **Market Unit** is the big antenna looing asset. <sup>(If it shows error item)</sup>
>The Market Pod is the tiny screen asset that you interact with.  <sup>(If it shows error item)</sup>

#### Setting up Market assets.

>[!NOTE]
>**This portion you will do in game.**

**Go into the Game** and 
1. **PLACE STATIC CORE " ANY SIZE "**

2. Place **"MARKET UNIT M "** inside previous placed Static Core.

3. Place  **" MARKET POD "**

4. Then link the **"MARKET POD"** to the **"MARKET UNIT M"**

5. RELOG <sup>( RESTART GAME OR JUST LOGOUT AND BACK INTO YOUR SERVER )</sup>

**You have now created a Market owned by your character.** 

#### HOW TO  CHANGE NAME OF  YOUR OWN MARKET OR NEW ONCES WHAT HAVE BEEN PLACED

>[!NOTE]
>There are two way to change the name of your player owned Market.

##### **The Easy Way**      

----------------------------------------------------------------

Login to Backoffice (BO)

Click on the **Markets Tab**

Scroll down until you find the new market you have just created. <sup>It will most likely be named **"My Market"** </sup>

Copy the "ID" of the created Market
![[Pasted image 20240822091952.png]]

Scroll Down to the **BATCH PARAMETER UPDATE** section at the bottom of the page.
![[Pasted image 20240822092335.png]]
Paste or type **Market ID** in the first filed outline in Red in this example

Then type the desired name for your market in the **Update Parameters** section in the **Name** field.

Then you need to apply your changes by clicking the **Update** button.

Now logout of the game client and log back in.
>[!TIP]
>If you have a server with a large player base **Restart the Server**

**DONE** 

> "HYYYYYPEEEEEEE WUPPPIIII DOOO"
> - GamingGothic

##### **Complicated Way**
----------------------------------------------------------------

1.  Open **CMD** and navigate the the server directory:
	`cd C:\Users\yourusername\Dual Universe Server`

3. Now paste this code via right clicking inside the CMD window: 
	`Docker-compose exec postgres psql -U postgres -d dual`

3. Now you will copy and paste behind "dual=#": 
>[!NOTE]
>THIS CODE  TO SEE ALL MARKETS

```Select * from market;``` 

4. Find your market in the list, SPAM SPACEBAR UNTIL BOTTOM

5. Look for **"MY MARKET"** and  get the **ID** and change the name in the following code:
> [!IMPORTANT]
> PLS READ THE CODE AND INSERT YOUR ID + WANTED NAME

`Update market set name ='INSERT NEW NAME HERE' where id = 'INSERT ID HERE';`

6. Run the command one time to check that the name of the market changed:
> [!IMPORTANT]
> REMOVE THE INSERTIDHERE AND ADD YOUR ID THERE FROM THE MARKET

`Select * from market where id = INSERTIDHERE;` 

7. Now logout of the game client and log back in.
>[!TIP]
>If you have a server with a large player base **Restart the Server**

>[!NOTE]
> JUST DO A SERVER RESTART SO EVERYONE WILL SEE THE MARKET.

----------------------------------------------------------------

**Credits:**
- twtich/Gaminggothic + twtichchat ! <3

**Twtichchat helper:**
- TheRealTnO 1
- M_M_MK     
- LordSaygon
- X_neckro_x
- Debtcollector 87

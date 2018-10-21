**Tehtävien deadline poikkeuksellisesti vasta maanantaina 22.10. klo 23.59**

Jatkamme sovelluksen rakentamista siitä, mihin jäimme viikon 6 lopussa. Allaoleva materiaali olettaa, että olet tehnyt kaikki edellisen viikon tehtävät. Jos et tehnyt kaikkia tehtäviä, voit täydentää ratkaisusi tehtävien palautusjärjestelmän kautta näkyvän esimerkivastauksen avulla.

## Muistutus debuggerista

Viikolla 2 tutustuimme [debuggeriin](https://github.com/mluukkai/WebPalvelinohjelmointi2018/blob/master/web/viikko2.md#debuggeri) ja [viime viikolla](https://github.com/mluukkai/WebPalvelinohjelmointi2018/blob/master/web/viikko6.md#muistutus-debuggerista) oli muistutus debuggerin käytösä. Edelleen edellisellä viikolla oli kuitenkin havaittavissa debuggerin alikäyttöä. Eli vielä kertauksena **kun kohtaat ongelman, turvaudu arvailun sijaan byebugiin!**

Rails-konsolin käytön tärkeyttä sovelluskehityksen välineenä on yritetty korostaa läpi kurssin. Eli **kun teet jotain vähänkin epätriviaalia, testaa asia ensin konsolissa.** Joissain tilanteissa voi olla jopa parempi tehdä kokeilut debuggerin avulla avautuvassa konsolissa, sillä tällöin on mahdollista avata konsolisessio juuri siihen kontekstiin, mihin koodia ollaan kirjoittamassa. Näin ollen päästään käsiksi esim. muuttujiin <code>params</code>, <code>sessions</code> ym. suorituskontekstista riippuvaan dataan.

## Testeistä

Osa tämän viikon tehtävistä saattaa hajottaa jotain edellisinä viikkoina tehtyjä testejä. Voit merkitä tehtävät testien hajoamisesta huolimatta, eli testien ja travisin pitäminen kunnossa on vapaaehtoista.

## Erilaiset järjestykset

Päätetään toteuttaa oluiden listalle toiminnallisuus, jonka avulla oluet voidaan järjestää eri sarakkeiden perusteella. Välitetään tieto halutusta järjestyksestä kontrollerille HTTP-pyynnön parametrina. Muutetaan näkymässä ```app/views/beers/index.html.erb``` olevaa taulukkoa seuraavasti:

```erb
<table class="table table-hover">
  <thead>
    <tr>
      <th> <%= link_to 'Name', beers_path(order:"name") %> </th>
      <th> <%= link_to 'Style', beers_path(order:"style") %> </th>
      <th> <%= link_to 'Brewery', beers_path(order:"brewery") %> </th>
    </tr>
  </thead>

  <tbody>
    <% @beers.each do |beer| %>
      <tr>
        <td><%= link_to beer.name, beer %></td>
        <td><%= link_to beer.style, beer.style %></td>
        <td><%= link_to beer.brewery.name, beer.brewery %></td>
      </tr>
    <% end %>
  </tbody>
</table>
```

eli taulukon sarakkeiden otsikoista on nyt tehty linkit, jotka johtavat takaisin samalle sivulle mutta lisäävät pyyntöön [query parametrin](https://en.wikipedia.org/wiki/Query_string) <code>:order</code>, joka määrittelee halutun järjestyksen. Käytännössä parametri välitetään urlin mukana liittäen se "normaalin" urlin perään kysymysmerkillä erottaen. Esim. jos klikataan tyylisaraketta, tulee urliksi _beers?order=style_

Kontrolleri pääsee HTTP-pyynnön parametriin käsiksi <code>params</code>-hashin avulla ja kuten olettaa saattaa, suunnan määrittelevän parametrin arvo on <code>params[:order]</code>.

Laajennetaan oluiden kontrolleria siten, että se testaa onko pyynnössä parametria, ja jos on, oluet järjestetään halutulla tavalla:

```ruby
def index
  @beers = Beer.all

  order = params[:order] || 'name'

  @beers = case order
    when 'name' then @beers.sort_by{ |b| b.name }
    when 'brewery' then @beers.sort_by{ |b| b.brewery.name }
    when 'style' then @beers.sort_by{ |b| b.style.name }
  end
end
```

Koodi määrittelee järjestämisen tapahtuvan oletusarvoisesti nimen perusteella. Tämä tapahtuu seuraavasti

```ruby
order = params[:order] || 'name'
```

Normaalisti <code>order</code> saa arvon <code>params[:order]</code>, jos parametria <code>:order</code> ei ole asetettu, eli sen arvo on <code>nil</code>, tulee arvoksi <code>||</code>:n jälkeinen osa eli 'name'.

**Huom1:** käytämme oluiden järjestämiseen rubyn <code>case when</code>-komentoa

```ruby
@beers = case order
  when 'name' then @beers.sort_by{ |b| b.name }
  when 'brewery' then @beers.sort_by{ |b| b.brewery.name }
  when 'style' then @beers.sort_by{ |b| b.style.name }
end
```

joka toimii oleellisesti samoin kuin seuraava

```ruby
  @beers =
  if order == 'name'
    @beers.sort_by{ |b| b.name }
  elsif orded == 'brewery'
    @beers.sort_by{ |b| b.brewery.name }
  elsif orded == 'style'
    @beers.sort_by{ |b| b.style.name }
  end
```

**Huom2:** esimerkissä oluet haetaan ensin tietokannasta ja sen jälkeen järjestetään ne keskusmuistissa. Oluiden lista olisi mahdollista järjestää myös tietokantatasolla:

```ruby
# oluet nimen perusteella järjestettynä
Beer.order(:name)

# oluet panimoiden nimien perusteella järjestettynä
Beer.includes(:brewery).order("breweries.name")

# oluet tyylin nimien perusteella järjestettynä
Beer.includes(:style).order("style.name")
```

> ## Tehtävä 1
>
> Muuta olutseurat listaavaa sivua siten, että seurat voidaan järjestää nimen mukaiseen aakkosjärjestykseen, perustamisvuoden mukaiseen järjestykseen tai kaupungin nimen mukaiseen aakkosjärjestykseen. Nimen mukainen järjestys on oletusarvoinen. 
>
> *HUOM* jos et ole toteuttanut sovellukseesi olutkerhoja, voit toteuttaa tämän tehtävän toiminnallisuuden panimoiden sivulle (ja olettaa että aktiiviset ja lopettaneet panimot järjestetään aina samalla tavalla).

## Selainpuolella toteutettu toiminnallisuus

Ratkaisumme oluiden listan järjestämiseen on melko hyvä. Suorituskyvyn kannalta hieman ongelmallista on tosin se, että aina järjestettäessä tehdään kutsu palvelimelle, joka generoi uudessa järjestyksessä näytettävän sivun.

Järjestämistoiminnallisuus voitaisiin toteuttaa myös selaimen puolella javascriptillä. Vaikka kurssi keskittyy palvelinpuolen toiminnallisuuteen, näytetään seuraavassa esimerkki siitä, miten järjestämistoiminnallisuus toteutettaisiin selainpuolella. Tässä ratkaisussa palvelin tarjoaa ainoastaan oluiden listan json-muodossa, ja selaimessa suoritettava javascript-koodi hoitaa myös oluet listaavan taulukon muodostamisen.

Emme korvaa nyt olemassaolevaa oluiden listaa, eli sivun beers toiminnallisuutta, sen sijaan tehdään toiminnallisuutta varten kokonaan uusi, osoitteessa beerlist toimiva sivu. Tehdään sivua varten reitti tiedostoon routes.rb:

    get 'beerlist', to:'beers#list'

Käytämme siis olutkontrollerissa olevaa <code>list</code>-metodia. Metodin ei tarvitse tehdä mitään:

```ruby
class BeersController < ApplicationController
  before_action :ensure_that_signed_in, except: [:index, :show, :list]
  # muut before_actionit enallaan

  def list
  end

  ...
end
```

**HUOM** lisäsimme metodin <code>list</code> niiden joita ennen ei tarvitse suorittaa <code>ensure_that_signed_in</code>-metodia, eli oluiden javascriptilla tuotetun listan näkeminen ei edellytä sivulle kirjautumista!

Myös näkymä views/beers/list.html.erb on minimalistinen:

```erb
<h2>Beers</h2>

<div id="beers"></div>
```

Eli näkymä ainoastaan sijoittaa sivulle div-elementin, jolle annetaan id:ksi (eli viitteksi, jolla elementtiin päästään käsiksi) "beers".

Kuten odotettua, osoitteessa http://localhost:3000/beerlist ei nyt näy mitään muuta kuin h2-elementin sisältö.

Alamme nyt kirjoittamaan toimintalogiikan toteutusta javascriptillä hyödyntäen [JQuery](https://jquery.com/)-kirjastoa.

Rails-sovelluksen tarvitsema javascript-koodi kannattaa sijoittaa hakemistoon app/assets/javascripts. Tehdään hakemistoon tiedosto _beerlist.js_ jolla on seuraava sisältö:

```javascript
document.addEventListener("turbolinks:load", () => {
  $('#beers').html("hello from javascript");
  console.log("hello console!");
})
```

Kun sivu nyt avataan uudelleen, asetetaan javascriptillä tai tarkemmin sanottuna jQuery-kirjastolla id:n <code>beers</code> omaavaan elementtiin teksti "hello form javascript". Seuraava komento kirjoittaa javascript-konsoliin tervehdyksen.

Javascript-ohjelmoinnissa selaimessa oleva konsoli on **erittäin tärkeä** työväline. Konsolin saa avattua chromessa tools-valikosta tai painamalla ctrl, shift, j (linux) tai alt, cmd, i (mac):

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2018/raw/master/images/ratebeer-w7-1.png)

**Konsoli on syytä pitää koko ajan auki Javascriptillä ohjelmoitaessa!**

Javascript näyttää aluksi melko kryptiseltä, mm. paljon käytettyjen anonyymifunktioiden takia. Edellä oleva koodi määrittelee että sivun lataututtua tapahtuvan __"turbolinks:load"-tapahtuman yhteydessä suoritetaan anonyymifunktion määrittelemä koodi:

```javascript
() => {
  $('#beers').html("hello from javascript");
  console.log("hello console!");
}
```


Jos kokeilemme selaimella osoitetta http://localhost:3000/beers.json huomaamme, että saamme vastaukseksi oluiden tiedot tekstuaalisessa json-muodossa (ks. http://en.wikipedia.org/wiki/JSON, http:www.json.org):

```ruby
[{"id":10,"name":"Extra Light Triple Brewed","style":{"id":1,"name":"European pale lager","description":"Similar to Munich Helles, many European countries reacted to the popularity of early pale lagers by brewing their own. Hop flavor is significant and of noble varieties, bitterness is moderate, and both are backed by a solid malt body and sweet notes from an all-malt base.","created_at":"2018-09-20T10:17:39.358Z","updated_at":"2018-09-20T10:35:04.921Z"},"brewery_id":1,"created_at":"2018-09-01T16:47:54.117Z","updated_at":"2018-09-20T10:17:39.414Z","url":"http://localhost:3000/beers/10.json"},{"id":6,"name":"Hefeweizen","style":{"id":4,"name":"German hefeweizen","description":"A south German style of wheat beer (weissbier) typically made with a ratio of 50 percent barley to 50 percent wheat. Sometimes the percentage of wheat is even higher. \"Hefe\" means \"with yeast,\" hence the beer's unfiltered and cloudy appearance. The particular ale yeast used produces unique esters and phenols of banana and cloves with an often dry and tart edge, some spiciness, and notes of bubblegum or apples. Hefeweizens are typified by little hop bitterness, and a moderate level of alcohol. Often served with a lemon wedge (popularized by Americans), to cut the wheat or yeasty edge, some may find this to be either a flavorful snap or an insult that can damage the beer's taste and head retention.","created_at":"2018-09-20T10:17:39.361Z","updated_at":"2018-09-20T10:36:17.788Z"},"brewery_id":3,"created_at":"2018-09-01T16:41:53.522Z","updated_at":"2018-09-20T10:17:39.406Z","url":"http://localhost:3000/beers/6.json"},{"id":7,"name":"Helles","style":{"id":1,"name":"European pale lager","description":"Similar to Munich Helles, many European countries reacted to the popularity of early pale lagers by brewing their own. Hop flavor is significant and of noble varieties, bitterness is moderate, and both are backed by a solid malt body and sweet notes from an all-malt base.","created_at":"2018-09-20T10:17:39.358Z","updated_at":"2018-09-20T10:35:04.921Z"},"brewery_id":3,"created_at":"2018-09-01T16:41:53.525Z","updated_at":"2018-09-20T10:17:39.408Z","url":"http://localhost:3000/beers/7.json"},{"id":16,"name":"Helles","style":{"id":1,"name":"European pale lager","description":"Similar to Munich Helles, many European countries reacted to the popularity of early pale lagers by brewing their own. Hop flavor is significant and of noble varieties, bitterness is moderate, and both are backed by a solid malt body and sweet notes from an all-malt base.","created_at":"2018-09-20T10:17:39.358Z","updated_at":"2018-09-20T10:35:04.921Z"},"brewery_id":3,"created_at":"2018-09-08T10:56:52.592Z","updated_at":"2018-09-20T10:17:39.420Z","url":"http://localhost:3000/beers/16.json"},{"id":4,"name":"Huvila Pale Ale","style":{"id":2,"name":"American Pale Ale","description":"Originally British in origin, this style is now popular worldwide and the use of local or imported ingredients produces variances in character from region to region. American versions tend to be cleaner and hoppier (with the piney, citrusy Cascade variety appearing frequently) than British versions, which are usually more malty, buttery, aromatic, and balanced. Pale Ales range in color from deep gold to medium amber. Fruity esters and diacetyl can vary from none to moderate, and hop aroma can range from lightly floral to bold and pungent. In general, expect a good balance of caramel malt and expressive hops with a medium body and a mildly bitter finish. ","created_at":"2018-09-20T10:17:39.359Z","updated_at":"2018-09-22T12:07:42.742Z"},"brewery_id":2,"created_at":"2018-09-01T16:41:53.516Z","updated_at":"2018-09-20T10:17:39.396Z","url":"http://localhost:3000/beers/4.json"},{"id":9,"name":"IVB","style":{"id":1,"name":"European pale lager","description":"Similar to Munich Helles, many European countries reacted to the popularity of early pale lagers by brewing their own. Hop flavor is significant and of noble varieties, bitterness is moderate, and both are backed by a solid malt body and sweet notes from an all-malt base.","created_at":"2018-09-20T10:17:39.358Z","updated_at":"2018-09-20T10:35:04.921Z"},"brewery_id":1,"created_at":"2018-09-01T16:46:01.643Z","updated_at":"2018-09-20T10:17:39.412Z","url":"http://localhost:3000/beers/9.json"},{"id":1,"name":"Iso 3","style":{"id":1,"name":"European pale lager","description":"Similar to Munich Helles, many European countries reacted to the popularity of early pale lagers by brewing their own. Hop flavor is significant and of noble varieties, bitterness is moderate, and both are backed by a solid malt body and sweet notes from an all-malt base.","created_at":"2018-09-20T10:17:39.358Z","updated_at":"2018-09-20T10:35:04.921Z"},"brewery_id":1,"created_at":"2018-09-01T16:41:53.508Z","updated_at":"2018-09-20T10:17:39.384Z","url":"http://localhost:3000/beers/1.json"},{"id":2,"name":"Karhu","style":{"id":1,"name":"European pale lager","description":"Similar to Munich Helles, many European countries reacted to the popularity of early pale lagers by brewing their own. Hop flavor is significant and of noble varieties, bitterness is moderate, and both are backed by a solid malt body and sweet notes from an all-malt base.","created_at":"2018-09-20T10:17:39.358Z","updated_at":"2018-09-20T10:35:04.921Z"},"brewery_id":1,"created_at":"2018-09-01T16:41:53.511Z","updated_at":"2018-09-20T10:17:39.389Z","url":"http://localhost:3000/beers/2.json"},{"id":8,"name":"Lite","style":{"id":1,"name":"European pale lager","description":"Similar to Munich Helles, many European countries reacted to the popularity of early pale lagers by brewing their own. Hop flavor is significant and of noble varieties, bitterness is moderate, and both are backed by a solid malt body and sweet notes from an all-malt base.","created_at":"2018-09-20T10:17:39.358Z","updated_at":"2018-09-20T10:35:04.921Z"},"brewery_id":1,"created_at":"2018-09-01T16:45:09.037Z","updated_at":"2018-09-20T10:17:39.410Z","url":"http://localhost:3000/beers/8.json"},{"id":14,"name":"Nanny State","style":{"id":6,"name":"Low alcohol beer","description":"Low Alcohol Beer is also commonly known as Non Alcohol (NA) beer, despite containing small amounts of alcohol. Low Alcohol Beers are generally subjected to one of two things: a controlled brewing process that results in a low alcohol content, or the alcohol is removed using a reverse-osmosis method which passes alcohol through a permeable membrane. They tend to be very light on aroma, body, and flavor.","created_at":"2018-09-20T10:17:39.362Z","updated_at":"2018-09-22T12:11:57.808Z"},"brewery_id":5,"created_at":"2018-09-06T14:30:50.585Z","updated_at":"2018-09-20T10:17:39.418Z","url":"http://localhost:3000/beers/14.json"},{"id":23,"name":"Panimomestarin IPA","style":{"id":5,"name":"American IPA","description":"Today's American IPA is a different soul from the IPA style first reincarnated in the 1980s. More flavorful and aromatic than the withering English IPA, its color can range from very pale golden to reddish amber. Hops are the star here, and those used in the style tend to be American with an emphasis on herbal, piney, and/or fruity (especially citrusy) varieties. Southern Hemisphere and experimental hops do appear with some frequency though, as brewers seek to distinguish their flagship IPA from a sea of competitors. Bitterness levels vary, but typically run moderate to high. Medium bodied with a clean, bready, and balancing malt backbone, the American IPA has become a dominant force in the marketplace, influencing brewers and beer cultures worldwide.","created_at":"2018-09-20T10:17:39.361Z","updated_at":"2018-09-22T12:09:23.686Z"},"brewery_id":1,"created_at":"2018-09-22T10:33:04.353Z","updated_at":"2018-09-22T10:33:04.353Z","url":"http://localhost:3000/beers/23.json"},{"id":13,"name":"Punk IPA","style":{"id":5,"name":"American IPA","description":"Today's American IPA is a different soul from the IPA style first reincarnated in the 1980s. More flavorful and aromatic than the withering English IPA, its color can range from very pale golden to reddish amber. Hops are the star here, and those used in the style tend to be American with an emphasis on herbal, piney, and/or fruity (especially citrusy) varieties. Southern Hemisphere and experimental hops do appear with some frequency though, as brewers seek to distinguish their flagship IPA from a sea of competitors. Bitterness levels vary, but typically run moderate to high. Medium bodied with a clean, bready, and balancing malt backbone, the American IPA has become a dominant force in the marketplace, influencing brewers and beer cultures worldwide.","created_at":"2018-09-20T10:17:39.361Z","updated_at":"2018-09-22T12:09:23.686Z"},"brewery_id":5,"created_at":"2018-09-06T14:30:33.589Z","updated_at":"2018-09-20T10:17:39.416Z","url":"http://localhost:3000/beers/13.json"},{"id":22,"name":"Sink the Bismarck","style":{"id":3,"name":"Baltic Porter","description":"Porters of the late 1700's were quite strong compared to today's standards, easily surpassing 7 percent alcohol by volume. Some English brewers made a stronger, more robust version, to be shipped across the North Sea that they dubbed a Baltic Porter. In general, the style's dark brown color covered up cloudiness and the smoky, roasted brown malts and bitter tastes masked brewing imperfections. Historically, the addition of stale ale also lent a pleasant acidic flavor to the style, which made it quite popular. These issues were quite important given that most breweries at the time were getting away from pub brewing and opening up production facilities that could ship beer across the world.","created_at":"2018-09-20T10:17:39.360Z","updated_at":"2018-09-22T12:08:13.953Z"},"brewery_id":5,"created_at":"2018-09-22T10:09:59.120Z","updated_at":"2018-09-22T10:09:59.120Z","url":"http://localhost:3000/beers/22.json"},{"id":21,"name":"Trans European Lager","style":{"id":1,"name":"European pale lager","description":"Similar to Munich Helles, many European countries reacted to the popularity of early pale lagers by brewing their own. Hop flavor is significant and of noble varieties, bitterness is moderate, and both are backed by a solid malt body and sweet notes from an all-malt base.","created_at":"2018-09-20T10:17:39.358Z","updated_at":"2018-09-20T10:35:04.921Z"},"brewery_id":1,"created_at":"2018-09-20T10:42:19.312Z","updated_at":"2018-09-20T10:42:19.312Z","url":"http://localhost:3000/beers/21.json"},{"id":3,"name":"Tuplahumala","style":{"id":1,"name":"European pale lager","description":"Similar to Munich Helles, many European countries reacted to the popularity of early pale lagers by brewing their own. Hop flavor is significant and of noble varieties, bitterness is moderate, and both are backed by a solid malt body and sweet notes from an all-malt base.","created_at":"2018-09-20T10:17:39.358Z","updated_at":"2018-09-20T10:35:04.921Z"},"brewery_id":1,"created_at":"2018-09-01T16:41:53.513Z","updated_at":"2018-09-20T10:17:39.392Z","url":"http://localhost:3000/beers/3.json"},{"id":5,"name":"X Porter","style":{"id":3,"name":"Baltic Porter","description":"Porters of the late 1700's were quite strong compared to today's standards, easily surpassing 7 percent alcohol by volume. Some English brewers made a stronger, more robust version, to be shipped across the North Sea that they dubbed a Baltic Porter. In general, the style's dark brown color covered up cloudiness and the smoky, roasted brown malts and bitter tastes masked brewing imperfections. Historically, the addition of stale ale also lent a pleasant acidic flavor to the style, which made it quite popular. These issues were quite important given that most breweries at the time were getting away from pub brewing and opening up production facilities that could ship beer across the world.","created_at":"2018-09-20T10:17:39.360Z","updated_at":"2018-09-22T12:08:13.953Z"},"brewery_id":2,"created_at":"2018-09-01T16:41:53.519Z","updated_at":"2018-09-20T10:17:39.400Z","url":"http://localhost:3000/beers/5.json"}]
```

Json-muotoisen sivun saa hieman luettavampaan muotoon esim. kopioimalla sivun sisällön [jsonlint](http://jsonlint.com/) palveluun:

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2018/raw/master/images/ratebeer-w7-3.png)

Parempi ratkaisu on asentaa selaimeen jsonia ymmärtävä plugin, eräs suositeltava on chromen [jsonview](https://chrome.google.com/webstore/detail/jsonview/chklaanhfefbnpoihckbnefhakgolnmc), plugin muotoilee jsonin selaimeen todella siististi:

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2017/raw/master/images/ratebeer-w7-4.png)

Tarkemmin tarkasteltuna jokainen yksittäinen json-muotoinen olut muistuttaa hyvin paljon Rubyn hashiä:

```ruby
{
  "id":10,"name":"Extra Light Triple Brewed",
  "style":{
    "id":1,"name":"European pale lager",
    "description":"Similar to Munich Helles, many European countries reacted to the popularity of early pale lagers by brewing their own. Hop flavor is significant and of noble varieties, bitterness is moderate, and both are backed by a solid malt body and sweet notes from an all-malt base.",
    "created_at":"2018-09-20T10:17:39.358Z",
    "updated_at":"2018-09-20T10:35:04.921Z"
  },
  "brewery_id":1,
  "created_at":"2018-09-01T16:47:54.117Z",
  "updated_at":"2018-09-20T10:17:39.414Z","url":"http://localhost:3000/beers/10.json"}
```

Minkä takia Rails osaa tarvittaessa palauttaa resurssit HTML:n sijaan jsonina?

Yritetään saada kaikkien reittausten lista jsonina, eli kokeillaan osoitetta http://localhost:3000/ratings.json

Seurauksena on virheilmoitus:

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2018/raw/master/images/ratebeer-w7-4b.png)

Eli ihan automaattisesti jsonit eivät synny, loimme kaiken reittaukseen liittyvän koodin käsin, ja kuten virheilmoituksesta voimme päätellä, formaatille 'json' ei ole olemassa sopivaa templatea.

Huomaamme, että scaffoldilla luotujen resurssien, esim. oluen views-hakemistosta löytyy joukko _json.jbuilder_-päätteisiä templateja, ja kuten arvata saattaa, käyttää Rails näitä jos resurssi halutaan json-muotoisena.

Ottamalla mallia templatesta app/views/beers/index.json.jbuilder teemme reittauksille seuraavan json.jbuilder-templaten (tiedosto on siis app/views/ratings/index.json.jbuilder):

```ruby
json.array!(@ratings) do |rating|
  json.extract! rating, :id, :score
end
```

ja nyt saamme reittaukset jsonina osoitteesta http://localhost:3000/ratings.json

```ruby
[{"id":31,"score":34},{"id":30,"score":42},{"id":27,"score":40},{"id":25,"score":12},{"id":24,"score":10}]
```

*HUOM:* jbuilder-templatessa käytettävän muuttujan <code>@ratings</code>
 tulee olla määritelty kontrollerin metodissa <code>index</code>!

Voisimme helposti määritellä json.jbuilder-templatessa, että reittausten json-esitykseen sisällytetään myös reittausta koskevan oluen tiedot:

```ruby
json.array!(@ratings) do |rating|
  json.extract! rating, :id, :score, :beer
end
```

Lisää jbuilderista seuraavissa http://railscasts.com/episodes/320-jbuilder?autoplay=true ja https://github.com/rails/jbuilder

Json-jbuilder-templatejen ohella toinen tapa palauttaa json-muotoista dataa olisi käytää <code>respond_to</code>-komentoa, jota muutamat scaffoldienkin generoivat metodit käyttävät. Tällöin json-jbuilder-templatea ei tarvittaisi ja kontrolleri näyttäisi seuraavalta

```ruby
def index
  @ratings = Rating.all

  respond_to do |format|
    format.html { } # renderöidään oletusarvoinen template
    format.json { render json: @ratings }
  end
end
```

Jbuilder-templatejen käyttö on kuitenkin ehdottomasti parempi vaihtoehto, tällöin json-muotoisen "näytön" eli resurssin representaation muodostaminen eriytetään täysin kontrollerista. Ei ole kontrollerin vastuulla muotoilla vastauksen ulkoasua oli kyseessä sitten json- tai HTML-muotoinen vastaus.

Palataan oluiden sivun pariin. Kun muodostamme sivun javascriptillä, ideana onkin hakea palvelimelta nimenomaan oluet json-muodossa ja renderöidä ne sitten sopivasti javascriptin avulla.

Muokataan javascript-koodiamme seuraavasti:

```javascript
document.addEventListener("turbolinks:load", () => {
  $.getJSON('beers.json', (beers) => {
    oluet = beers
    $("#beers").html("oluita löytyi " + beers.length);
  });
})
```

Koodin ensimmäinen rivi (joka siis suoritetaan heti kun sivu on latautunut) tekee HTTP GET -kyselyn palvelimen osoitteeseen beers.json ja määrittelee takaisinkutsufunktion, jota selain kutsuu siinä vaiheessa kun GET-kyselyyn tulee vastaus palvelimelta. Takaisinkutsufunktion parametrissa <code>beers</code> on palvelimelta tullut data, eli json-muodossa oleva oluiden lista. Muuttujan sisältö sijoitetaan globaaliin muuttujaan <code>oluet</code> ja oluiden listan pituus näytetään www-sivulla id:n beers omaavassa elementissä.

Koska sijoitimme viitteen oluiden listan globaaliin muuttujaan, voimme tarkastella sitä selaimen konsolista käsin (muistutuksena että konsolin saa avattua chromessa tools-valikosta tai painamalla ctrl, shift, j (linux) tai alt, cmd, i (mac) ja jos olit jo sulkenut konsolin teit pahan virheen. **Konsoli tulee pitää aina auki javascriptillä ohjelmoitaessa!**):

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2018/raw/master/images/ratebeer-w7-2.png)

Takaisinkutsufunktion pitäisi siis saatuaan oluet palvelimelta muodostaa ne listaava HTML-koodi ja lisätä se sivulle.

Muutetaan javascript-koodiamme siten, että se listaa aluksi ainoastaan oluiden nimet:

```javascript
$.getJSON('beers.json', (beers) => {
  const beer_list = []

  beers.forEach( (beer) => {
    beer_list.push('<li>' + beer['name'] + '</li>')
  })

  $("#beers").html('<ul>'+ beer_list.join('') + '</ul>')
});
```

Koodi määrittelee paikallisen taulukkomuuttujan <code>beer_list</code> ja käy läpi parametrina saamansa oluiden listan <code>beers</code> ja lisää jokaista olutta kohti <code>beer_list</code>:iin HTML-elementin, joka on muotoa

```erb
<li>Extra Light Triple Brewed</li>
```

Lopuksi listan alkuun ja loppuun lisätään ul-tagit ja listan alkiot liitetään yhteen join-metodilla. Näin saatu HTML-koodi liitetään id:n <code>beers</code> omaavaan elementtiin.

Nyt siis saimme yksinkertaisen listan oluiden nimistä sivulle.

Entä jos haluaisimme järjestää oluet? Jotta tämä onnistuu, refaktoroimme koodin ensin seuraavanlaiseksi:

```javascript
const BEERS = {}

BEERS.show = () => {
  const beer_list = []

  BEERS.list.forEach((beer) => {
    beer_list.push('<li>' + beer['name'] + '</li>')
  })

  $("#beers").html('<ul>' + beer_list.join('') + '</ul>')
}

document.addEventListener("turbolinks:load", () => {
  $.getJSON('beers.json', (beers) => {
    BEERS.list = beers
    BEERS.show()
  })
})
```

Määrittelimme nyt olion <code>BEERS</code>, jonka attribuuttiin <code>BEERS.list</code> palvelimelta saapuva oluiden lista sijoitetaan. Metodi <code>BEERS.show</code> muodostaa <code>BEERS.list</code>:in oluista HTML-taulukon ja sijoittaa sen näytölle.

Näin muotoiltuna palvelimelta haettu oluiden lista jää "muistiin" selaimeen muuttujaan <code>BEERS.list</code> ja lista voidaan tarpeen tullen uudelleenjärjestää ja näyttää käyttäjälle uudessa järjestyksessä ilman että www-sivun tarvitsee ollenkaan kommunikoida palvelimen kanssa.

Lisätään sivulle painike (tai linkki), jota painamalla oluet saadaan sivulle käänteiseen järjestykseen:

```erb
<a href="#" id="reverse">reverse!</a>
<div id="beers"></div>
```

Lisätään sitten javascriptillä linkille klikkauksenkäsittelijä, joka linkkiä klikatessa laittaa oluet käänteiseen järjestykseen ja näyttää ne sivun beers-elementissä:

```javascript
const BEERS = {}

BEERS.show = () => {
  const beer_list = []

  BEERS.list.forEach((beer) => {
    beer_list.push('<li>' + beer['name'] + '</li>')
  })

  $("#beers").html('<ul>' + beer_list.join('') + '</ul>')
}

BEERS.reverse = () => {
  BEERS.list.reverse()
}

document.addEventListener("turbolinks:load", () => {
  $("#reverse").click((e) => {
    e.preventDefault()
    BEERS.reverse()
    BEERS.show()
  })

  $.getJSON('beers.json', (beers) => {
    BEERS.list = beers
    BEERS.show()
  })
})
```

Linkin klikkauksen käsittelijä siis määritellään sivun latautumista vastaavan tapahtuman <code>turbolinks:load</code> sisällä, eli kun dokumentti on latautunut, _rekisteröidään_ klikkausten käsittelijäfunktio id:n "reverse" omaavalle linkkielementille.  

Kun linkkiä klikataan, tapahtumankäsittelijä kutsuu aluksi metodia <code>e.preventDefault</code>, joka estää klikkauksen "normaalin" toiminnallisuuden eli (nyt olemattoman) linkin seuraamisen. Tämän jälkeen kutsutaan metodeita _reverse_ ja _show_ piirtämään oluet ruudulle käänteisessä järjestysessä.

Nyt ymmärrämme riittävästi perusteita ja olemme valmiina toteuttamaan todellisen toiminnallisuuden.

Muutetaan näkymää seuraavasti:

```erb
<h2>Beers</h2>

<table id="beertable" class="table table-hover">
  <tr>
    <th> <a href="#" id="name">Name</a> </th>
    <th> <a href="#" id="style">Style</a> </th>
    <th> <a href="#" id="brewery">Brewery</a> </th>
  </tr>

</table>
```

Eli kolmesta sarakenimestä on tehty linkki, joihin tullaan rekisteröimään klikkauksenkuuntelijat. Taulukolle on annettu id <code>beertable</code>.

Muutetaan sitten javascriptissä määriteltyä metodia <code>show</code> siten, että se laittaa oluiden nimet taulukkoon:

```javascript
BEERS.show = () => {
  const table = $("#beertable")

  BEERS.list.forEach((beer) => {
    table.append('<tr><td>' + beer['name'] + '</td></tr>')
  })
}
```

Eli ensin koodi tallettaa viitteen taulukkoon muuttujana <code>table</code> ja lisää sinne <code>append</code>-komennolla uuden rivin kutakin olutta varten.

Laajennetaan sitten metodia näyttämään kaikki tiedot oluista. Huomaamme kuitenkin, että oluiden json-muotoisessa listassa <http://localhost:3000/beers.json> ei ole panimosta muuta tietoa kuin olioiden id:t, haluaisimme kuitenkin näyttää panimon nimen. Oluttyylin tiedot löytyvät kokonaisuudessaan jsonista jo nyt.

Ongelma on onneksi helppo ratkaista muokkaamalla oluiden listan tuottavaa json-jbuildertemplatea. Template näyttää nyt seuraavalta:

```ruby
json.array! @beers, partial: 'beers/beer', as: :beer
```

Template määrittelee, että jokaisesta oluesta muodostetaan json-esitys tiedoston *_beer.json.jbuilder* avulla, tiedoston sisältö on seuraava

```ruby
json.extract! beer, :id, :name, :style, :brewery_id, :created_at, :updated_at
json.url beer_url(beer, format: :json)
```

Tiedosto määrittelee, että yksittäisen oluen jsoniin sisällytetään kentät _id_, _name_ ja _brewery_id_ sekä _style_ joka taas viittaa olueeseen liittyvään <code>Style</code>-olioon. Tyyliolio tuleekin renderöityä oluen json-esityksen sisälle kokonaisuudessaan. Saamme myös panimon json-esityksen oluen jsonin mukaan jos korvaamme templatessa _brewery_id_:n _brewery_:llä. Muutamme siis yksittäisen oluen jsonin renderöinnistä vastaavan templaten seuraavaan muotoon:

```ruby
json.extract! beer, :id, :name, :style, :brewery
```

poistimme viimeisen rivin joka lisäsi jokaisen oluen json-esityksen mukaan urlin oluen omaan json-esitykseen, poistimme myös aikaleimakentät.

Nyt saamme taulukon generoitua seuraavalla javascriptillä:

```javascript
BEERS.show = () => {
  const table = $("#beertable")

  BEERS.list.forEach((beer) => {
    table.append('<tr>'
      + '<td>' + beer['name'] + '</td>'
      + '<td>' + beer['style']['name'] + '</td>'
      + '<td>' + beer['brewery']['name'] + '</td>'
      + '</tr>')
  })
}
```

Oluiden listan json-esityksen mukana tulee nyt paljon tarpeetontakin tietoa sillä mukaan renderöityvät jokaisen oluen panimon ja tyylin json-esitykset kokonaisuudessaan. Voisimme optimoida yksittäisen oluen json-esityksen generoivaa templatea siten, että oluen panimosta ja tyylistä tulee json-esitykseen mukaan ainoastaan nimi:

```ruby
json.extract! beer, :id, :name
json.style do
  json.name beer.style.name
end
json.brewery do
  json.name beer.brewery.name
end
```

Nyt palvelimen lähettämä oluiden jsonmuotoinen lista on huomattavasti inhimillisemmän kokoinen:

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2018/raw/master/images/ratebeer-w7-5.png)

Rekisteröimme vielä järjestämisen suorittavat tapahtumankuuntelijat linkeille (seuraavassa lopullinen javascript-koodi):

```javascript
const BEERS = {}

BEERS.show = () => {
  $("#beertable tr:gt(0)").remove()
  const table = $("#beertable")

  BEERS.list.forEach((beer) => {
    table.append('<tr>'
      + '<td>' + beer['name'] + '</td>'
      + '<td>' + beer['style']['name'] + '</td>'
      + '<td>' + beer['brewery']['name'] + '</td>'
      + '</tr>')
  })
}

BEERS.sort_by_name = () => {
  BEERS.list.sort((a, b) => {
    return a.name.toUpperCase().localeCompare(b.name.toUpperCase());
  })
}

BEERS.sort_by_style = () => {
  BEERS.list.sort((a, b) => {
    return a.style.name.toUpperCase().localeCompare(b.style.name.toUpperCase())
  })
}

BEERS.sort_by_brewery = () => {
  BEERS.list.sort((a, b) => {
    return a.brewery.name.toUpperCase().localeCompare(b.brewery.name.toUpperCase());
  })
}

document.addEventListener("turbolinks:load", () => {
  $("#name").click((e) => {
    e.preventDefault()
    BEERS.sort_by_name()
    BEERS.show();
    
  })

  $("#style").click((e) => {
    e.preventDefault()
    BEERS.sort_by_style()
    BEERS.show()
  })

  $("#brewery").click((e) => {
    e.preventDefault()
    BEERS.sort_by_brewery()
    BEERS.show()
  })

  $.getJSON('beers.json', (beers) => {
    BEERS.list = beers
    BEERS.show()
  })
})
```

Javascript-koodimme tulee liitetyksi sovelluksen jokaiselle sivulle. Tästä on se ikävä seuraus, että ollaanpa millä sivulla tahansa, lataa javascript oluiden listan komennon <code>getJSON('beers.json', ...) </code> takia. Myös tapahtumakunntelijat yritetään rekisteröidä jokaiselle sivulle vaikka niiden rekisteröinti on mielekästä ainoastaan jos ollaan oluiden listalla.

Viritellään javascript-koodia vielä siten, että <code>turbolinks:load</code>:n tapahtumankäsittelijässä oleva koodi suoritetaan ainoastaan jos ollaan sivulla, josta taulukko <code>beertable</code> löytyy:

```javascript
document.addEventListener("turbolinks:load", () => {
  if ($("#beertable").length == 0) {
    return
  } 
  
  $("#name").click((e) => {
    e.preventDefault()
    BEERS.sort_by_name()
    BEERS.show();
    
  })

  $("#style").click((e) => {
    e.preventDefault()
    BEERS.sort_by_style()
    BEERS.show()
  })

  $("#brewery").click((e) => {
    e.preventDefault()
    BEERS.sort_by_brewery()
    BEERS.show()
  })

  $.getJSON('beers.json', (beers) => {
    BEERS.list = beers
    BEERS.show()
  })
})
```

Tällä hetkellä trendinä siirtää yhä suurempi osa web-sivujen toiminnallisuudesta selaimeen. Etuna mm. se että web-sovelluksien toiminta saadaan muistuttamaan yhä enenevissä määrin desktop-sovelluksia.

## React 

Äsken javascriptillä toteuttamamme oluet listaava sivu oli koodin rakenteen puolesta ihan kohtuullista, mutta Railsin sujuvuuteen ja vaivattomuuteen verrattuna koodi oli raskaahkoa ja paikoin ikävien, rutiininomaisten yksityiskohtien täyttämää. Jos sovelluksen selainpuolella toteutettavan koodin määrä alkaa kasvaa, on lopputuloksena helposti sekava koodi, jonka toiminnasta kukaan ei enää ota selvää ja jonka laajentaminen muuttuu erittäin haastavaksi.

Javascript-frontendsovelluskehykset tuovat asiaan helpotusta. Tämän hetken suosituin ratkaisu frontendien tekemiseen on Facebookin kehittämä [React](https://facebook.github.io/react/). React on laaja aihe ja pääset syventymään siihen laitoksen kurssilla Fullstack-websovellusohjelmointi joka järjestetään nyt menossa olevana [avoimen yliopiston kurssina](https://fullstackopen.github.io/) ja uudistettuna versiona [periodissa 3](https://courses.helsinki.fi/fi/TKT21009/124960978).

> ## Tehtävä 2
>
> Toteuta edellisten esimerkkien tyyliin javascriptillä kaikki panimot listaava sivu http:localhost:3000/brewerylist  
>
> Sivulla näytetään jokaisesta panimosta nimi, perustusvuosi, panimon valmistamien oluiden lukumäärä ja tieto siitä onko panimo lopettanut. Sivun siis **ei** tarvitse eritellä lopettaneita panimoita omaan taulukkoonsa.
>
> Panimoiden järjestäminen toteutetaan vasta seuraavassa tehtäässä.
>
> **Muista pitää Javascript-konsoli koko ajan auki tehtävää tehdessäsi!** Voit debugata Javasriptia tulostelemalla konsoliin komennolla <code>console.log()</code>
>
> **HUOM:** edellisellä viikolla tekmämme muutoksen takia panimoiden json-lista http://localhost:3000/breweries.json ei toimi, sillä breweries#index-kontrolleri ei enää aseta kaikkien panimoiden listaa muuttujaan <code>@breweries</code>. Korjaa tilanne.
>
> **HUOM2:** tehtävä kannattaa tehdä yksi pieni askel kerrallaan, samaan tapaan kuin oluiden lista tehtiin yllä olevassa esimerkissä. Javascriptin debuggaus saattaa olla haasteellista ja **varmin tapa aiheuttaa iso turhautuma onkin yrittää tehdä tehtävä nopeasti copypasteamalla beerlistin koodi**.

> ## Tehtävä 3
>
> Laajenna panimoiden listaa siten, että panimot voi järjestää joko aakkos- tai perustamisvuoden mukaiseen järjestykseen tai panimon valmistamien oluiden lukumäärän perusteella. 

## Selainpuolella toteutetun toiminnallisuuden testaaminen

Tehdään rspec/capybaralla muutama testi javascriptillä toteutetulle oluiden listalle. Seuraavassa on lähtökohtamme, tiedosto spec/features/beerlist_page_spec.rb:

```ruby
require 'rails_helper'

describe "Beerlist page" do
  before :all do
    Capybara.register_driver :selenium do |app|
      Capybara::Selenium::Driver.new(app, :browser => :chrome)
    end
  end

  before :each do
    @brewery1 = FactoryBot.create(:brewery, name:"Koff")
    @brewery2 = FactoryBot.create(:brewery, name:"Schlenkerla")
    @brewery3 = FactoryBot.create(:brewery, name:"Ayinger")
    @style1 = Style.create name:"Lager"
    @style2 = Style.create name:"Rauchbier"
    @style3 = Style.create name:"Weizen"
    @beer1 = FactoryBot.create(:beer, name:"Nikolai", brewery: @brewery1, style:@style1)
    @beer2 = FactoryBot.create(:beer, name:"Fastenbier", brewery:@brewery2, style:@style2)
    @beer3 = FactoryBot.create(:beer, name:"Lechte Weisse", brewery:@brewery3, style:@style3)
  end

  it "shows one known beer" do
    visit beerlist_path
    expect(page).to have_content "Nikolai"
  end
end
```

Suoritetaan testi komennolla <code>rspec spec/features/beerlist_page_spec.rb</code>. Tuloksena on kuitenkin virheilmoitus:

```ruby
  1) Beerlist page shows one known beer
     Failure/Error: expect(page).to have_content "Nikolai"
       expected to find text "Nikolai" in "breweries beers styles ratings users clubs places signin signup\nyou should be signed in\nSign in\nusername password"
     # ./spec/features/beerlist_page_spec.rb:18:in `block (2 levels) in <top (required)>'

Finished in 21.17 seconds (files took 5.33 seconds to load)
1 example, 1 failure
```

Näyttää siis siltä että sivulla ei ole ollenkaan oluiden listaa. Varmistetaan tämä laittamalla testiin juuri ennen komentoa <code>expect</code> komento <code>save_and_open_page</code> jonka avulla saamme siis avattua selaimeen sivun jolle capybara on navigoinut
(ks. https://github.com/mluukkai/WebPalvelinohjelmointi2018/blob/master/web/viikko4.md#capybarav4#capybara).

Ja aivan kuten arvelimme, sivulla näytettävä oluttaulukko on tyhjä:

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2018/raw/master/images/ratebeer-w6-2.png)

Syy ongelmalle löytyy capybaran dokumentaatiosta https://github.com/jnicklas/capybara#drivers

> By default, Capybara uses the :rack_test driver, which is fast but limited: it does not support JavaScript, nor is it able to access HTTP resources outside of your Rack application, such as remote APIs and OAuth services. To get around these limitations, you can set up a different default driver for your features.

Ja korjauskin on helppo. Javascriptiä tarvitseviin testeihin riittää lisätä parametri, jonka ansiosta testi suoritetaan javascriptiä osaavan Selenium-testiajurin avulla:

```ruby
it "shows the known beers", js:true do
```

Kun suoritamme testit, törmäämme virheilmoitukseen

```ruby
1) Beerlist page shows one known beer
    Failure/Error: visit beerlist_path

    WebMock::NetConnectNotAllowedError:
      Real HTTP connections are disabled. Unregistered request: GET http://127.0.0.1:52187/__identify__ with headers {'Accept'=>'*/*', 'Accept-Encoding'=>'gzip;q=1.0,deflate;q=0.6,identity;q=0.3', 'User-Agent'=>'Ruby'}

      You can stub this request with the following snippet:

      stub_request(:get, "http://127.0.0.1:52187/__identify__").
        with(
          headers: {
        'Accept'=>'*/*',
        'Accept-Encoding'=>'gzip;q=1.0,deflate;q=0.6,identity;q=0.3',
        'User-Agent'=>'Ruby'
          }).
        to_return(status: 200, body: "", headers: {})

      ============================================================
```

Virheen syy on siinä, että otimme viikolla 5 käyttöömme [WebMock-gemin](https://github.com/mluukkai/WebPalvelinohjelmointi2017/blob/master/web/viikko5.md#olutpaikkojen-etsimistoiminnon-testaaminen) joka oletusarvoisesti kieltää testikoodin suorittamat HTTP-yhteydet. Javascriptilla toteutettu olutlistahan yrittää hakea oluiden listan json-muodossa palvelimelta. Pääsemme virheestä eroon sallimalla yhteydet paikalliselle palvelimelle, esim. muuttamalla testit alustavaan <code>before :all</code> -lohkoa seuraavasti:

```ruby
before :all do
  Capybara.register_driver :selenium do |app|
    Capybara::Selenium::Driver.new(app, :browser => :chrome)
  end
  WebMock.disable_net_connect!(allow_localhost: true) 
end
```

Testi toimii vihdoin.


Kun sivuille luodaan sisältöä javascriptillä, ei sisältö ilmesty sivulle vielä samalla hetkellä kuin sivun html-pohja ladataan vaan vasta javascript takaisinkutsufunktion suorituksen jälkeen. Eli jos katsomme sivun sisältöä välittömästi sivulle navigoinnin jälkeen, ei javascript ole vielä ehtinyt muodostaa sivun lopullista sisältöä. Esim. seuraavassa <code>save_and_open_page</code> saattaa avata sivun, jossa ei vielä näy yhtään olutta:

``` ruby
it "shows a known beer", js:true do
  visit beerlist_path
  save_and_open_page
  expect(page).to have_content "Nikolai"
end
```

Kuten sivulla https://github.com/jnicklas/capybara#asynchronous-javascript-ajax-and-friends sanotaan, osaa capybara odottaa asynkroonisia javascript-kutsuja sen verran, että testien sivulta etsimät elementit ovat latautuneet.

Tiedämme, että javascriptin pitäisi lisätä sivun taulukkoon rivejä. Saammekin sivun näkymään oikein, jos lisäämme alkuun komennon <code>find('table').find('tr:nth-child(2)')</code> joka etsii sivulta taulukon ja sen sisältä toisen rivin (taulukon ensimmäinen rivihän on jo sivupohjassa mukana oleva taulukon otsikkorivi):

``` ruby
it "shows a known beer", :js => true do
  visit beerlist_path
  find('table').find('tr:nth-child(2)')
  save_and_open_page
  expect(page).to have_content "Nikolai"
end
```

Nyt capybara odottaa taulukon valmistumista ja siirtyy sivun avaavaan komentoon vasta taulukon latauduttua (itseasiassa vain 2 riviä taulukkoa on varmuudella valmiina).

Testien suorittaminen todellisessa selaimella on melko hidasta. Saat nopeutettua testejä käyttämällä Chromen Headless- moodia, eli "käyttöliittymätöntä versiota". Headless-selaimen käyttöönotto onnistuu muuttamalla <code>before :all</code> -lohko muotoon 

``` ruby
before :all do
  Capybara.register_driver :selenium do |app|
    capabilities = Selenium::WebDriver::Remote::Capabilities.chrome(
      chromeOptions: { args: ['headless', 'disable-gpu']  }
    )

    Capybara::Selenium::Driver.new app,
      browser: :chrome,
      desired_capabilities: capabilities      
  end
  WebMock.disable_net_connect!(allow_localhost: true) 
end
```

Konfiguraation muutoksen jälkeen suoritus normaalilla selaimella onnistuu poistamalla parametri <code>headless</code> kolmannelta riviltä.

> ## Tehtävä 4
>
> Tee testi joka varmistaa, että oluet ovat beerlist-sivulla oletusarvoisesti nimen mukaan aakkosjärjestyksessä
>
> Testaaminen kannattaa tehdä nyt siten, että etsitään taulukon rivit <code>find</code>-selektorin avulla ja varmistetaan, että jokaisella rivillä on oikea sisältö. Koska taulukossa on otsikkorivi, löytyy ensimmäinen varsinainen rivi seuraavasti:
>
> ``` ruby
> find('table').find('tr:nth-child(2)')
> ``` 
>
> Rivin sisältöä voi testata normaaliin tapaan expect ja have_content -metodeilla.

> ## Tehtävä 5
>
> Tee testit seuraaville toiminnallisuuksille
> * klikattaessa saraketta 'style' järjestyvät oluet tyylin nimen mukaiseen aakkosjärjestykseen
> * klikattaessa saraketta 'brewery' järjestyvät oluet panimon nimen mukaiseen aakkosjärjestykseen

**Huom.** Travis ei osaa suoraan ajaa Selenium-testejä. Ongelmaan löytyy vastaus täältä  http://about.travis-ci.org/docs/user/gui-and-headless-browsers/#Using-xvfb-to-Run-Tests-That-Require-GUI-(e.g.-a-Web-browser)
Travisin toimintaansaattaminen muutosten jälkeen on vapaaehtoista.

## Asset pipeline

Rails-sovelluksiin liittyviä javascript- ja tyylitiedostoja (ja kuvia) hallitaan ns. Asset pipelinen avulla, ks. https://guides.rubyonrails.org/asset_pipeline.html

Periaatteena on se, että sovelluskehittäjä sijoittaa sovellukseen liittyvät javascript-tiedostot hakemistoon _app/assets/javascripts_ ja tyylitiedostot hakemistoon _app/assets/stylesheets_.  Molempia voidaan sijoittaa useaan eri tiedostoon, ja tarvittaessa alihakemistoihin.

Sovellusta kehitettäessä (eli kun sovellus on ns. development-moodissa) Rails liittää kaikki (ns. manifest-tiedostossa) määritellyt javascript- ja tyylitiedostot mukaan sovellukseen. Huomaammekin tarkastellessamme sovellusta selaimen view source -ominaisuuden avulla, että mukaan on liitetty suuri joukko javascriptiä ja tyylitiedostoja.

Sovelluksen mukaan liitettävät javascript-tiedostot määritellään tiedostossa _app/assets/javascripts/application.js_, jonka sisältö on nyt seuraava

```javascript
//= require rails-ujs
//= require activestorage
//= require turbolinks
//= require_tree .
//= require jquery3
//= require popper
//= require bootstrap-sprockets
```

Vaikka koko tiedoston sisältö näyttää olevan kommenteissa, on kuitenkin kyse "oikeista", asset pipelinestä huolehtivan [sprockets-kääntäjän](https://github.com/sstephenson/sprockets) komennoista, joiden avulla määritellään sovellukseen mukaan otettavat javascript-tiedostot. Tiedosto määrittelee, että mukaan otetaan rails-ujs, activestorage, turbolinks, jquery3, popper ja bootstrap-sprockets. Kaikki näistä on asennettu sovellukseen gemien avulla.

Rivi <code>//= require_tree .</code> määrittelee, että kaikki hakemiston *assets/javascripts/* ja sen alihakemistojen sisältämät javascript-tiedostot sisällytetään ohjelmaan.

Asset pipeline mahdollistaa myös [coffeescriptin](http://coffeescript.org/) käyttämisen, tällöin tiedostojen päätteeksi tulee <code>.js.coffee</code>. Sovellusta suoritettaessa scprockets kääntää coffeescriptin automaattisesti javascriptiksi. 

Tuotantokäytössä sovelluksella ei suorituskykysyistä yleensä kannata olla useampia javascript- tai tyylitiedostoja. Kun sovellusta aletaan suorittaa tuotantoympäristössä (eli production-moodissa), sprockets yhdistääkin kaikki sovelluksen javascript- ja tyylitiedostot yksittäisiksi, optimoiduiksi tiedostoiksi. Huomaamme tämän jos katsomme herokussa olevan sovelluksen html-lähdekoodia, esim: http://wad-ratebeer.herokuapp.com/ sisältää se nyt ainoastaan yhden js- ja yhden css-tiedoston joista varsinkin js-tiedoston luettavuus on ihmisen kannalta heikko.

Lisää asset pipelinestä ja mm. javascriptin liittämisestä railssovelluksiin mm. seuraavissa:
* http://railscasts.com/episodes/279-understanding-the-asset-pipeline
* http://railsapps.github.io/rails-javascript-include-external.html

> ## Tehtävät 6-8 (kolmen tehtävän arvoinen)
>
> ### Tehtävä on hieman työläs, joten tee ensin helpommat pois alta. Muut viikon tehtävät eivät riipu tästä tehtävästä.
>
> Toistaiseksi kuka tahansa voi sovelluksessamme liittyä olutkerhon jäseneksi. Muutetaan nyt sovellusta siten, että jäsenyys ei tule voimaan ennenkuin joku jo jäsenenä oleva vahvistaa jäsenyyden.
>
> Muutamia huomioita
> * jäsenyyden vahvistamattomuus kannattaa huomioida siten, että Membership-modeliin lisätään boolean-arvoinen kenttä _confirmed_
> * Kun kerho luodaan, tee sen luoneesta käyttäjästä automaattisesti kerhon jäsen
> * Näytä kerhon sivulla jäsenille lista vahvistamattomana olevista jäsenyyksistä (eli jäsenhakemuksista)
> * Jäsenyyden statuksen muutos voidaan hoitaa esim. oman [custom-reitin](https://github.com/mluukkai/WebPalvelinohjelmointi2017/blob/master/web/viikko6.md#reitti-panimon-statuksen-muuttamiselle) avulla.
>
> Tehtävä saattaa olla hieman haastava. [Active Record Associations -guiden](http://guides.rubyonrails.org/association_basics.html) luku **4.3.3 Scopes for has_many** tarjoaa erään hyvän työvälineen tehtävään. Tehtävän voi toki tehdä monella muullakin tavalla.
> Myös luku **4.3.2.3 :class_name** voi olla hyödyksi.

Tehtävän jälkeen sovelluksesi voi näyttää esim. seuraavalta. Olutseuran sivulla näytetään lista jäsenyyttä hakeneista, jos kirjautuneena on olutseurassa jo jäsenenä oleva käyttäjä:

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2017/raw/master/images/ratebeer-w6-6.png)

Käyttäjän omalla sivulta näytetään toistaiseksi käsittelemättömät hakemukset:

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2017/raw/master/images/ratebeer-w6-5.png)

## Indeksi tietokantaan

Kun käyttäjä kirjautuu järjestelmäämme, suoritetaan sessiokontrollerissa operaatio, jossa käyttäjäolio haetaan tietokannasta käyttäjän nimen perusteella:

```ruby
class SessionsController < ApplicationController
  def create
    user = User.find_by username: params[:username]

     # ...
  end

end
```

Operaation suorittamista varten tietokanta joutuu käymään läpi koko <code>users</code>-taulun. Haut olion id:n suhteen ovat nopeampia, sillä jokainen taulu on indeksöity id:iden suhteen. Indeksi toimii hajautustaulun tavoin, eli tarjoaa "O(1)"-ajassa toimivan pääsyn haettuun tietokannan riviin.

Tietokantojen tauluihin voidaan lisätä tarvittaessa muitakin indeksejä. Nopeutetaan <code>users</code>-taulusta tapahtuvaa käyttäjätunnuksen perusteella tehtävää hakua lisäämällä taululle indeksi.

Luodaan indeksiä varten migraatio

    rails g migration AddUserIndexBasedOnUsername

Migraatio on seuraavanlainen:

```ruby
class AddConfirmedToMembership < ActiveRecord::Migration[5.2]
  def change
    add_index :users, :username
  end
end
```

Suoritetaan migraatio komennolla <code>rake db:migrate</code> ja indeksi on valmis!

Indeksin huono puoli on se, että kun järjestelmään lisätään uusi käyttäjä tai olemassaoleva käyttäjä poistetaan, on indeksiä muokattava ja tähän luonnollisestsi kuluu aikaa. Indeksin lisäys on siis tradeoff sen suhteen, mitä operaatiota halutaan optimoida.

## Laiska lataaminen, n+1-ongelma ja tietokantakyselyjen optimointi

Kaikki oluet näyttävä kontrolleri on yksinkertainen. Oluet haetaan tietokannasta, järjestetään HTTP-kutsussa olleen parametrin määrittelemällä tavalla ja asetetaan templatea varten muuttujaan:

```ruby
def index
  @beers = Beer.all

  order = params[:order] || 'name'

  @beers = case order
            when 'name' then @beers.sort_by(&:name)
            when 'brewery' then @beers.sort_by{ |b| b.brewery.name }
            when 'style' then @beers.sort_by{ |b| b.style.name }
            end
end
```

Template listaa oluet taulukkona:

```erb
<% @beers.each do |beer| %>
  <tr>
    <td><%= link_to beer.name, beer %></td>
    <td><%= link_to beer.style, beer.style %></td>
    <td><%= link_to beer.brewery.name, beer.brewery %></td>
  </tr>
<% end %>
</table>
```

Yksinkertaista ja tyylikästä... mutta ei kovin tehokasta.


Voisimme katsoa lokitiedostosta log/development.log mitä kaikkea oluiden sivulle mentäessä tapahtuu. Pääsemme samaan tietoon hieman mukavammassa muodossa käsiksi _miniprofiler_ gemin (ks. https://github.com/MiniProfiler/rack-mini-profiler ja http://samsaffron.com/archive/2012/07/12/miniprofiler-ruby-edition)

Miniprofilerin käyttöönotto on helppoa, riittää että Gemfileen lisätään rivi

    gem 'rack-mini-profiler'

Suorita <code>bundle install</code> ja käynnistä rails server uudelleen. Kun menet tämän jälkeen osoitteeseen http:localhost:300/beers huomaat, että sivun yläkulmaan ilmestyy aikalukema joka kuvaa HTTP-pyynnön suoritukseen käytettyä aikaa. Numeroa klikkaamalla avautuu tarkempi erittely ajankäytöstä:

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2017/raw/master/images/ratebeer-w6-7.png)

Raportti kertoo että <code>Executing action: index</code> eli kontrollerimetodin suoritus aiheuttaa yhden SQL-kyselyn <code>SELECT "beers".* FROM "beers"</code>. Sen sijaan
 <code>Rendering: beers/index</code> eli näkymätemplaten suoritus aiheuttaa peräti 11 SQL-kyselyä!

Kyselyjä klikkaamalla päästään tarkastelemaan syytä:

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2017/raw/master/images/ratebeer-w6-8.png)

Näkymätemplaten renderöinti siis suorittaa useaan kertaan seuraavat kyselyt:

```ruby
SELECT  "styles".* FROM "styles" WHERE "styles"."id" = ? LIMIT ?;

SELECT  "breweries".* FROM "breweries" WHERE "breweries"."id" = ? LIMIT ?; 
```
Käytännössä jokaista erillistä olutta kohti tehdään oma kysely sekä <code>styles</code>- että <code>breweries</code> tauluun.

Syynä tälle on se, että activerecordissa on oletusarvoisesti käytössä ns. _lazy loading_, eli kun haemme olion tietokannasta, olioon liittyvät kentät haetaan tietokannasta vasta jos niihin viitataan. Joskus tämä käyttäytyminen on toivottavaa, olioonhan voi liittyä suuri määrä olioita, joita ei välttämättä tarvita olion itsensä käsittelyn yhteydessä. Kaikkien oluiden sivulle mentäessä lazy loading ei kuitenkaan ole hyvä idea, sillä tiedämme varmuudella että jokaisen oluen yhteydessä näytetään myös oluen panimon sekä tyylin nimet ja nämä tiedot löytyvät ainoastaan panimoiden ja tyylien tietokantatauluista.

Voimme ohjata ActiveRecordin metodien parametrien avulla kyselyistä generoituvaa SQL:ää. Esim. seuraavasti voimme ohjeistaa, että oluiden lisäksi niihin liittyvät panimot tulee hakea tietokannasta:

```ruby
def index
  @beers = Beer.includes(:brewery).all
  # ...
end
```

Miniprofilerin avulla näemme. että kontrollerin suoritus aiheuttaa nyt kaksi kyselyä:

```ruby
SELECT "beers".* FROM "beers"; 
SELECT "breweries".* FROM "breweries" WHERE "breweries"."id" IN (?, ?, ?, ?);
```

Näyttötemplaten suoritus aiheuttaa enää 7 kyselyä, jotka kaikki ovat muotoa:

```ruby

SELECT  "styles".* FROM "styles" WHERE "styles"."id" = ? LIMIT ?; 
```

Näytön renderöinnin yhteydessä enää on haettava oluisiin liittyvät tyylit tietokannasta, kukin omalla SQL-kyselyllä.

Optimoidaan kontrolleria vielä siten, että myös kaikki tarvittavat tyylit luetaan kerralla kannasta:

```ruby
def index
  @beers = Beer.includes(:brewery, :style).all

  # ...
end
```

Kontrollerin suoritus aiheuttaa nyt kolme kyselyä ja näytön renderöinti ainoastaan yhden kyselyn. Miniprofiler paljastaa että kysely on

```ruby
SELECT  "users".* FROM "users"  WHERE "users"."id" = ? LIMIT 1
```

ja syynä sille on

```ruby
app/controllers/application_controller.rb:12:in `current_user'
```
eli näytön muuttujan <code>current_user</code> avulla tekemä viittaus kirjautuneena olevaan käyttäjään. Tämä ei kuitenkaan ole hirveän vakavaa.

Saimme siis optimoitua SQL-kutsujen määrän _1 + 2n_:stä (missä n tietokannassa olevien oluiden määrä) kolmeen (plus yhteen)! Määrä on sikäli hyvä, että se on vakio ja ei riipu järjestelmässä olevien oluiden määrästä.

Kokemaamme kutsutaan n+1-ongelmaksi (ks. http://guides.rubyonrails.org/active_record_querying.html#eager-loading-associations), eli hakiessamme kannasta yhdellä kyselyllä listallisen olioita, jokainen listan olioista aiheuttaakin salakavalasti uuden tietokantahaun ja näin yhden haun sijaan tapahtuukin noin n+1 hakua.

Muutetaan seuraavaa tehtävää varten kaikki käyttäjät listaava template  muotoon

```ruby
<h1>Users</h1>

<table class="table table-hover">
  <thead>
    <tr>
      <th>Username</th>
      <th> rated beers </th>
      <th> total ratings </th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <% @users.each do |user| %>
      <tr>
        <td><%= link_to user.username, user %></td>
        <td><%= user.beers.size %></td>
        <td><%= user.ratings.size %></td>
        <td>
          <% if admin_user and user.frozen? %>
            <span class="label label-info">account frozen</span>
          <% end %>
        </td>
      </tr>
    <% end %>
  </tbody>
</table>
```

Huomaa, että elementin <code>td</code> sisällä olevan if:in ehdon toimivuus riippuu siitä miten olet nimennyt asioita viikolla 5 tehdyn tehtävän koodissa. Voit tarvittaessa poistaa koko ehdon.

> ## Tehtävä 9
>
> Muutos aiheuttaa n+1-ongelman käyttäjien sivulle. Korjaa ongelma edellisen esimerkin tapaan eager loadaamalla tarvittavat oliot käyttäjien hakemisen yhteydessä. Varmista optimointisi onnistuminen miniprofilerilla.

**Huom:** jos taulukkoon liitettäisi myös suosikkioluen kertova sarake

     <td><%= user.favorite_beer %></td>

Muuttuisi tilanne hieman hankalammaksi SQL:n optimoinnin suhteen. Metodimme viimeisin versio oli seuraava:

```ruby
def favorite_beer
  return nil if ratings.empty?
  ratings.order(score: :desc).limit(1).first.beer
end
```

Nyt edes eager loadaaminen ei auta, sillä metodikutsu auheuttaa joka tapauksessa SQL-kyselyn. Jos sen sijaan toteuttaisimme metodin keskusmuistissa olueeseen liittyviä reittauksia (kuten teimme aluksi viikolla 4):

```ruby
def favorite_beer
  return nil if ratings.empty?
  ratings.sort_by{ |r| r.score }.last.beer
end
```

metodikutsu _ei_ aiheuttaisi tietokantaoperaatiota _jos_ reittaukset olisi eager loadattu siinä vaiheessa kun metodia kutsutaan.

Saattaakin olla, että metodista olisi tietyissä tilanteissa suorituskykyä optimoitaessa hyvä olla kaksi versiota, toinen joka suorittaa operaation tietokantatasolla ja toinen keskusmuistissa operaation tekevä.

## Cachays eli palvelinpuolen välimuistitoiminnallisuudet

Luodaan tietokantaamme hiukan lisää dataa. 

Korvaa tiedoston _db/seeds.db_ sisältö seuraavalla:

```ruby
users = 400             # jos koneesi on hidas, riittää esim 200
breweries = 200         # jos koneesi on hidas, riittää esim 100
beers_in_brewery = 50
ratings_per_user = 30

(1..users).each do |i|
  User.create! username: "user_#{i}", password:"Passwd1", password_confirmation: "Passwd1"
end

(1..breweries).each do |i|
  Brewery.create! name:"Brewery_#{i}", year: 1900, active: true
end

bulk = Style.create! name: "Bulk", description: "cheap, not much taste"

Brewery.all.each do |b|
  n = rand(beers_in_brewery)
  (1..n).each do |i|
    beer = Beer.create! name:"Beer #{b.id} -- #{i}", style:bulk
    b.beers << beer
  end
end

User.all.each do |u|
  n = rand(ratings_per_user)
  beers = Beer.all.shuffle
  (1..n).each do |i|
    r = Rating.new score:(1+rand(50))
    beers[i].ratings << r
    u.ratings << r
  end
end
```

Käytämme tiedostossa normaalien olioiden luovien metodien <code>create</code> sijaan huutomerkillistä versiota <code>create!</code>. Metodien erona on niiden käyttäytyminen tilanteessa, jossa olion luominen ei onnistu. Huutomerkitön metodi palauttaa tällöin arvon <code>nil</code>, huutomerkillinen taas aiheuttaa poikkeuksen. Seedauksessa poikkeuksen aiheuttaminen on parempi vaihtoehto, muuten luomisen epäonnistuminen jää herkästi huomaamatta.

**Kopioi sitten vanha tietokanta _db/development.sqlite_ talteen**, jotta voit palata vanhaan tilanteeseen suorituskyvyn virittelyn jälkeen. Vot ottaa vanhan tietokannan käyttöön muuttamalla sen nimeksi jälleen development.sqlite

**Huom:** tämä ei ole välttämättä paras mahdollinen tapa tehdä suorituskykytestausta oikeille Rails-sovelluksille, ks. lisää tietoa seuraavasta http://guides.rubyonrails.org/v3.2.13/performance_testing.html (guidesta ei ole Rails 5:lle päivitettyä versiota.)

Suorita seedaus komennolla

    rails db:reset

Skriptin suorittamisessa kuluu tovi.

**Huom:** jos skriptin suoritus päättyy virheeseen, kannattaa vian korjaamisen jälkeen palauttaa vanha tietokanta ennen skriptin uutta suorittamista. Eräs potentiaalinen ongelma skriptin suorituksessa on validoinnin rikkovat duplikaattinimet. Jos muutat komennon <code>create!</code> muotoon <code>create</code> ei skriptin suoritus keskeydy.

Nyt tietokannassamme on runsaasti dataa ja sivujen lataaminen alkaa olla hitaampaa.

Kokeile nyt miten sivujen suorituskykyyn vaikuttaa jos kommentoit pois äsken tekemäsi SQL-kyselyjen optimoinnit oluiden sivulta, eli muuta olutkontrolleri takaisin muotoon:

```ruby
def index
  # @beers = Beer.includes(:brewery, :style).all
  @beers = Beer.all

  order = params[:order] || 'name'

  @beers = case order
    when 'name' then @beers.sort_by{ |b| b.name }
    when 'brewery' then @beers.sort_by{ |b| b.brewery.name }
    when 'style' then @beers.sort_by{ |b| b.style.name }
  end
end
```

Huomioi myös suoritettujen SQL-kyselyjen määrä optimoinnilla ja ilman. Tuloksenhan näet jälleen kätevästi miniprofilerin avulla.

Kokeilun jälkeen voit palauttaa koodin optimoituun muotoon.

Datamäärän ollessa suuri, ei pelkkä kyselyjen optimointi riitä, vaan on etsittävä muita keinoja.

Vaihtoehdoksi nousee tällöin **cachaus eli välimuistien käyttö**.

Web-sovelluksessa cachaystä voidaan suorittaa sekä selaimen, että palvelimen puolella (sekä selaimen ja palvelimen välissä olevissa proxyissä). Tarkastellaan nyt palvelimen puolella tapahtuvaa cachaystä. Toteutimme jo [toissa viikolla](https://github.com/mluukkai/WebPalvelinohjelmointi2017/blob/master/web/viikko5.md#suorituskyvyn-optimointi) "käsin" beermapping-apista haettujen tietojen cachaystä Rails.cachen avulla. Tutkitaan nyt railsin tarjoamaa hieman automaattisempaa cachaysmekanismia.

Cachays ei ole oletusarvoisesti päällä kun sovellusta suoritetaan development-moodissa. Kytkit ehkä cachen päälle viikolla 5. 

Jos cache on päällä, on projektissa olemassa tiedosto _tmp/caching-dev.txt_. Jos tiedostoa ei ole, saat cachen päälle suorittamalla komentoriviltä komennon <code>rails dev:cache</code>. Komennon pitäisi tulostaa

```
Development mode is now being cached.
```

Jos komento tulostaa

```
Development mode is no longer being cached.
```
suorita se uudelleen.

**Käynnistä nyt sovellus uudelleen.**

Päätetään cachata oluiden listan näyttäminen.

Cachays tapahtuu sisällyttämällä näkymätemplaten cachattavan osa, eli _sivufragmentti_ seuraavanlaiseen lohkoon:

```erb
<% cache 'avain', skip_digest: true do %>
  cachättävä näkymätemplaten osa
<% end %>
```

Kuten arvata saattaa, <code>avain</code> on avain, jolla cachattava näkymäfragmentti talletetaan. Avaimena voi olla merkkijono tai olio. <code>skip_digest: true</code> liittyy [näyttötemplatejen versiointiin](http://blog.remarkablelabs.com/2012/12/russian-doll-caching-cache-digests-rails-4-countdown-to-2013) jonka haluamme nyt jättää huomioimatta. Tämä kuitenkin tarkoittaa, että välimuisti on syytä tyhjentää (komennolla <code>Rails.cache.clear</code>) _jos_ näkymätemplaten koodia muutetaan.

Fragmentticachayksen lisääminen oluiden listalle views/beers/index.html on helppoa, cachataan sivulta sen dynaaminen osa eli oluiden taulukko:

```erb
<h1>Beers</h1>

<% cache 'beerlist', skip_digest: true do %>

  <table class="table table-hover">
    <thead>
      <tr>
        <th> <%= link_to 'Name', beers_path(order:"name") %> </th>
        <th> <%= link_to 'Style', beers_path(order:"style") %> </th>
        <th> <%= link_to 'Brewery', beers_path(order:"brewery") %> </th>
      </tr>
    </thead>

    <tbody>
      <% @beers.each do |beer| %>
        <tr>
          <td><%= link_to beer.name, beer %></td>
          <td><%= link_to beer.style.name, beer.style %></td>
          <td><%= link_to beer.brewery.name, beer.brewery %></td>
        </tr>
      <% end %>
    </tbody>
  </table>

  <% end %>

<br>

<%= link_to('New Beer', new_beer_path, class:'btn btn-primary') if current_user %>
```

Kun nyt menemme sivulle, ei sivufragmenttia ole vielä talletettu välimuistin ja sivun lataaminen kestää yhtä kauan kuin ennen cachayksen lisäämistä:

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2017/raw/master/images/ratebeer-w6-9.png)

Sivun latausaika on 852.3, mistä itse sivun renderöimiseen kulunut aika <code>Rendering: beers/index</code> on siis 365.1 millisekuntia.

Sivulla käytyämme sivun osa tallettuu välimuistiin ja seuraava sivun avaaminen on huomattavasti nopeampi:

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2017/raw/master/images/ratebeer-w6-10.png)

Koko sivun latausaika on 154 millisekuntia, josta ainoastaan 4.5 millisekuntia kuluu näkymätemplaten lataamiseen.

Huom: uuden oluen luomislinkkiä ei kannata laittaa cachatyn fragmentin sisälle, sillä linkki tulee näyttää ainoastaan kirjautuneille käyttäjille. Sivun cachatty osa näytetään nyt kaikille samanlaisena.

Jos luomme nyt uuden oluen huomaamme, että uuden oluen tietoja ei tule sivulle. Syynä tälle on tietenkin se, että sivufragmentti löytyy edelleen välimuistista. Vanhentunut näkymäfragmentti tulisi siis ekspiroida. Tässä tapauksessa helpoin strategia on kontrollerista käsin tapahtuva manuaalinen ekspirointi.

Ekspirointi tapahtuu komennolla <code>expire_fragment(avain)</code> jota tulee siis kutsua kontrollerista niissä kohdissa joissa oluiden listan sisältö mahdollisesti muuttuu. Tälläisiä kohtia ovat olutkontrollerin metodit <code>create</code>, <code>update</code> ja <code>destroy</code>. Muutos on helppo:

```erb
def create
  expire_fragment('beerlist')
end

def update
  expire_fragment('beerlist')
  # ...
end

def destroy
  expire_fragment('beerlist')
  # ...
end
```

Muutosten jälkeen sivu toimii odotetulla tavalla!

Kaikkien oluiden sivua olisi mahdollista nopeuttaa vielä jonkin verran. Nyt nimittäin kontrolleri suorittaa tietokantaoperaation

```erb
@beers = Beer.includes(:brewery, :style).all
```

myös silloin kun sivufragmentti löytyy välimuistista. Voisimmekin testata fragmentin olemassaoloa metodilla <code>fragment_exist?</code> ja suorittaa tietokantaoperaation ainoastaan jos fragmentti ei ole olemassa:

```erb
def index
  # jos fragmentti olemassa, lopetetaan metodi tähän (eli renderöidään heti näkymä)
  return if request.format.html? && fragment_exist?('beerlist')

  @beers = Beer.includes(:brewery, :style).all

  order = params[:order] || 'name'

  @beers = case order
            when 'name' then @beers.sort_by(&:name)
            when 'brewery' then @beers.sort_by{ |b| b.brewery.name }
            when 'style' then @beers.sort_by{ |b| b.style.name }
            end
end
```

Ehdossa oleva <code>request.format.html?</code> varmistaa sen, että suoritamme kontrollerimetodin kaiken koodin siinä tapauksessa jos muodostamme _json_-muotoisen vastauksen.

Sivu nopeutuu entisestään:

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2017/raw/master/images/ratebeer-w6-11.png)

Huomaamme kuitenkin että sivulla on pieni ongelma. Oluet sai järjestettyä sarakkeita klikkaamalla vaihtoehtoisiin järjestyksiin. Cachays on kuitenkin rikkonut toiminnon!

Yksi tapa korjata toiminnallisuus on liittää fragmentin avaimeen järjestys:

```erb
<% cache "beerlist-#{@order}", skip_digest: true do %>
  taulukon html
<% end %>
```

Järjestys talletetaan siis muuttujaan <code>@order</code> kontrollerissa. Seuraavassa kontrollerin vaatimat muutokset:

```ruby
def index
  @order = params[:order] || 'name'
  return if request.format.html? && fragment_exist?("beerlist-#{@order}")

  @beers = Beer.includes(:brewery, :style).all
  @beers = case @order
            when 'name' then @beers.sort_by(&:name)
            when 'brewery' then @beers.sort_by{ |b| b.brewery.name }
            when 'style' then @beers.sort_by{ |b| b.style.name }
            end
end
```

Ekspiroinnin yhteydessä on ekspiroitava kaikki kolme järjestystä:

```ruby
["beerlist-name", "beerlist-brewery", "beerlist-style"].each{ |f| expire_fragment(f) }
```

**Huom:** voit kutsua fragmentticachen operaatioita konsolista:

```ruby
> ActionController::Base.new.fragment_exist?('beerlist-name')
Exist fragment? views/beerlist-name (0.4ms)
=> true
> ActionController::Base.new.expire_fragment('beerlist-name')
Expire fragment views/beerlist-name (0.6ms)
=> true
> ActionController::Base.new.fragment_exist?('beerlist-name')
Exist fragment? views/beerlist-name (0.1ms)
=> nil
```

> ## Tehtävä 10
>
> Toteuta panimot listaavalle sivulle fragmentticachays. Varmista, että sivun sisältöön vaikuttava muutos ekspiroi cachen. 

## yksittäisen oluen sivun cashays

Jos haluaisimme cachata yksittäisen oluen sivun, kannattaa fragmentin avaimeksi laittaa itse cachattava olio:

```ruby
<% cache @beer do %> 

  <h2>
    <%= @beer.name %>
  </h2>

  <p>
    <strong>Style:</strong>
    <%= link_to @beer.style.name, @beer.style %>
  </p>

  <p>
    <strong>Brewery:</strong>
    <%= link_to @beer.brewery.name, @beer.brewery %>
  </p>

  <p>
    <% if @beer.ratings.empty? %>
      beer has not yet been rated!
    <% else %>
      Has <%= pluralize(@beer.ratings.count, 'rating') %>, 
      average <%= round(@beer.average_rating) %>
    <% end %>
  </p>

<% end %>

<!- cachaamaton osa ->

<% if current_user %>
  <h4>give a rating:</h4>

  <%= form_for(@rating) do |f| %>
    <%= f.hidden_field :beer_id %>
    score: <%= f.number_field :score %>
    <%= f.submit class:"btn btn-primary" %>
  <% end %>
<% end %>

<%= edit_and_destroy_buttons(@beer) %>
```

Nyt fragmentin avaimeksi tulee merkkijono, jonka Rails generoi kutsumalla olion metodia <code>cache_key_with_version</code>. Metodi generoi avaimen joka yksilöi olion _ja_ sisältää aikaleiman, joka edustaa hetkeä, jolloin olio on viimeksi muuttunut. Jos olion kenttiin tulee muutos, muuttuu fragmentin avaimen arvo eli vanha fragmentti ekspiroituu automaattisesti. Seuraavassa esimerkki automaattisesti generoituvasta cache-avaimesta:

```ruby
> b = Beer.first
> b.cache_key_with_version
=> "beers/1-20180924183300410080"
> b.update_attribute(:name, 'ISO 4')
> b.cache_key_with_version
=> "beers/1-20180924183314873407"
```

Ratkaisu on vielä sikäli puutteellinen, että jos olueeseen tehdään uusi reittaus, olio ei itsessään muutu ja fragmentti ei ekspiroidu. Ongelma on kuitenkin helppo korjata. Lisätään reittaukseen tieto, että reittauksen syntyessä, muuttuessa tai tuhoutuessa, on samalla 'kosketettava' reittaukseen liittyvää olutta:

```ruby
class Rating < ApplicationRecord
  belongs_to :beer, touch: true

  # ...
end
```

Käytännössä <code>belongs_to</code>-yhteyteen liitetty <code>touch: true</code> saa aikaan sen, että yhteyden toisessa päässä olevan olion kenttä <code>updated_at</code> päivittyy.

> ## Tehtävä 11
>
> Toteuta yksittäisen panimon sivulle fragmentticachays. Huomaa, että edellisen esimerkin tapaan panimon sivufragmentin on ekspiroiduttava automaattisesti jos panimon oluisiin tulee muutoksia.

Välimuistin eksplisiittinen ekspiroiminen, kuten kaikkien oluiden sivun suhteen joudumme tekemään, on hieman ikävää sillä on aina pieni riski, että koodissa ei muisteta ekspiroida fragmenttia kaikissa tarpeellisissa kohdissa.

Käyttäessämme suoraan olioa (kuten yksittäisen oluen sivulla tehtiin) fragmentin avaimena, ekspiroitui cache automaattisesti olion päivittyessä. Myös kaikkien oluiden sivulle olisi mahdollista tehdä automaattisesti ekspiroituva cache generoimalla fragmentin avain tarkoitukseen sopivan metodin avulla, katso
[Caching with Rails: An overview](http://guides.rubyonrails.org/caching_with_rails.html#fragment-caching) -guiden
kohta "If you want to avoid expiring the fragment manually..."

## Selainpuolen cachays

Cachaysta harjoitetaan monilla tasoilla, myös selainpuolella. Lisää tietoa Railsin tuesta selainpuolen cachaykseen [täällä](https://guides.rubyonrails.org/caching_with_rails.html#conditional-get-support) 

## Eventual consistency

Sovelluksen käyttäjän kannalta ei ole aina välttämätöntä, että sovelluksen näyttämä tilanne on täysin ajantasalla. Esim. ei ole kriittistä, jos reittausstatistiikkaa näyttävä ratings-sivu näyttää muutaman minuutin vanhan tilanteen, pääasia on, että kaikki järjestelmään tullut data tulee ennen pitkää näkyville käyttäjille. Tälläisestä hieman löyhemmästä ajantasaisuusvaatimuksesta, jolla saatetaan pystyä tehostamaan sovelluksen suorituskykyä huomattavasti käytetään englanninkielistä nimitystä [eventual consistency](http://en.wikipedia.org/wiki/Eventual_consistency).

Eventual consistency -mallin mukainen ajantasaisuus on helppo määritellä Railsissa laittamalla esim. fragmentticachelle expiroitumisaika:

```erb
<% cache 'fragment_name', expires_in:10.minutes do %>
  ...
<% end %>
```

Tämä yksinkertaistaa sovellusta myös siinä mielessä, että cachen ekspiroiminen on helppoa, kun ei ole pakko huomioida kaikkia yksittäisiä kohtia koodissa, jotka voisivat aiheuttaa sivun epäajantasaisuuden.

Oletetaan, että järjestelmällämme olisi todella paljon käyttäjiä ja reittauksia tapahtuisi useita kertoja minuutissa. Jos haluaisimme näyttää ratings-sivulla täysin ajantasaista tietoa, ei sivun suorituskyky olisi hyvä, sillä jokainen oluen reittaus muuttaa sivun tilaa, ja sivu tulisi ekspiroida erittäin usein. Tämä taas tekisi cachayksestä lähes hyödyttömän.

SQL:n optimointi ja cachayskään eivät vielä tee ratings-sivusta kovin nopeita, sillä kontrollerin käyttämät operaatiot esim. <code>User.top(3)</code> vaativat käytännössä melkein koko tietokannan datan läpikäyntiä. Jos haluaisimme optimoida sivua vielä enemmän, tulisi meidän käyttää järeämpiä keinoja. Esim <code>User.top</code>-komennon suoritus nopeutuisi huomattavasti, jos käyttäjän reittausten määrä talletettaisiin suoraan käyttäjä-olioon, eli sen laskeminen ei edellyttäisi käyttäjään liittyvien Rating-olioiden lukumäärän laskemista. Tämä taas edellyttäisi, että aina käyttäjän uuden reittauksen yhteydessä päivitettäisiin myös käyttäjä-olioa. Eli itse reittausoperaation suoritus hidastuisi hieman.

Toinen ja ehkä parempi tapa reittaussivun nopeuttamiselle olisi cachata Rails.cacheen kontrollerin tarvitsemat tiedot. Kontrolleri on siis seuraava

```ruby
def index
  @ratings = Rating.recent
  @beers = Beer.top(3)
  @styles = Style.top(3)
  @breweries = Brewery.top(3)
  @users = User.top(3)
end
```

Voisimme toimia nyt samoin kuin viikolla 5 kun talletimme olutravintoloiden tietoja Railsin cacheen, eli kontrolleri muuttuisi suunnilleen seuraavaan muotoon:

```ruby
def index
  Rails.cache.write("beer top 3", Beer.top(3)) if cache_does_not_contain_data_or_it_is_too_old
  @top_beers = Rails.cache.read "beer top 3"

  # ...
end
```

Sovellusten suorituskyvyn optimointi ei ole välttämättä helppoa, se edellyttää monentasoisia ratkaisuja ja pitää useimmiten tehdä tilannekohtaisesti räätälöiden. Koodi muuttuu yleensä optimoinnin takia rumemmaksi.

## Asynkronisuus, viestijonot ja taustatyöt

Yhtenä negatiivisena puolena cachen ajoittain tapahtuvassa ekspiroimisessa, esim. jos noudattaisimme strategiaa ratings-sivun suhteen, aiheutuu jollekin käyttäjälle aika ajoin paljon aikaavievä operaatio siinä vaiheessa kun data on generoitava uudelleen välimuistiin.

Parempaan ratkaisuun päästäisiinkin jos käyttäjälle tarjottaisiin aina niin ajantasainen data kuin mahdollista, eli kontrolleri olisi muotoa:

```ruby
def index
  @top_beers = Rails.cache.read("beer top 3")

  # ...
end
```

Välimuistin päivitys voitaisiin sitten suorittaa omassa taustalla olevassa, aika ajoin heräävässä säikeessä/prosessissa:

```ruby
# pseudokoodia, ei toimi oikeasti...
def background_worker
  while true do
      sleep 10.minutes
      Rails.cache.write("beer top 3", Beer.top(3))
      Rails.cache.write("brewery top 3", Brewery.top(3))
      # ...
  end
end
```

Ylläesitellyn kaltainen taustaprosessointitapa on siinä mielessä yksinkertainen, että sovelluksen ja taustaprosessointia suorittavan säikeen/prosessin ei tarvitse synkronoida toimintojaan. Toisinaan taas taustaprosessoinnin tarpeen laukaisee jokin sovellukselle tuleva pyyntö. Tällöin sovelluksen ja taustaprosessoijien välisen synkronoinnin voi hoitaa esim. viestijonojen avulla.

Viestijonoilla ja erillisillä prosesseilla tai säikeillä hoidetun taustaprosessoinnin toteuttamiseen Railsissa on paljon erilaisia vaihtoehtoja, tämän hetken paras ratkaisu näistä on [Sidekiq](http://railscasts.com/episodes/366-sidekiq). 

Jos sovellus tarvitsee ainoastaan jonkin yksinkertaisen, tasaisin aikavälein suoritettavan taustaoperaation, saattaa [Heroku scheduler](https://devcenter.heroku.com/articles/scheduler) olla yksinkertaisin vaihtoehto. Tällöin taustaoperaatio määritellään [Rake-taskina](http://railscasts.com/episodes/66-custom-rake-tasks), jonka Heroku suorittaa joko kerran vuorokaudessa, tunnissa tai kymmenessä minuutissa.

## Sucker Punch

Kuten edellä todettiin, paras vaihtoehto asynkronisten operaatioiden suorittamiseen Railsilla on [Sidekiq](http://railscasts.com/episodes/366-sidekiq). Sidekiq kuitenkin vaatii oman prosessinsa, eli esim. Herokussa sidekiqia ei ole helppoa suorittaa varaamatta sille omaa prosessia eli [dynoa](https://devcenter.heroku.com/articles/dynos), ja se taas maksaa vähintään 7 dollaria kuussa.

Ilmaisten Heroku-palveluiden yhteydessä on madollista [Sucker Punch](https://github.com/brandonhilkert/sucker_punch)- kirjastoa:

> Sucker Punch is a single-process Ruby asynchronous processing library. This reduces costs of hosting on a service like Heroku along with the memory footprint of having to maintain additional jobs if hosting on a dedicated server. All queues can run within a single application (eg. Rails, Sinatra, etc.) process.

Eli Sucker Punch suorittaa asynkroniset työt samassa prosessissa, missä itse Rails-sovellustakin suoritetaan. 

Sucker Punchin käyttö on melko helppoa. 

Lisää gemfileen <code>gem 'sucker_punch', '~> 2.0'</code> ja suorita bundle install.


eli määritellään Rails lataamaan automaattisesti luomaamme hakemistoon määritelty koodi.

Luodaan nyt Sucker Punch -operatio, eli tiedosto _test_job.rb_ jolla on seuraava sisältö:

```ruby
class TestJob
  include SuckerPunch::Job

  def perform
    puts "running job..."
  end
end
```

Voimme suorittaa operaation antamalla rails-konsolista (tai mistä tahansa kohtaa sovelluksen koodia) komennon

```ruby
TestJob.perform_async
```

Operaatio tulostaa konsoliin _running job..._. Ei kovin vakuuttavaa.

Huomionarvoista tässä on kuitenkin se, että operaatio suoritetaan asynkronisesti taustalla, eli kontrolli palaa konsoliin jo ennen kuin operaatio on suoritettu.

Muutetaan operaatiota seuraavasti:

```ruby
class TestJob
  include SuckerPunch::Job

  def perform
    sleep 1
    puts "starting job..."
    sleep 10
    puts "job ready!"
  end
end
```

eli nyt operaation suoritus kestää 11 sekuntia. Kun suoritat operaation komennolla <code>TestJob.perform_async</code> huomaat, että pääset takaisin konsoliin välittömästi komennon suorituksen jälkeen (joudut todennäköisesti painamaan enteriä että saat konsolin komentokehotteen näkyviin) ja operaation suoritus tapahtuu taustalla, samalla kun voit suorittaa konsolista halutessasi jotain muuta koodia.

Voit suorittaa operaation myös _synkronisesti_ antamalla komennon <code>TestJob.new.perform</code>. Tällöin joudut odottamaan komennon suorituksen loppuun asti ennen kuin konsoli aktivoituu uudelleen.

Voit myös suorittaa operaatioita ajastetusti, esim. jos annat komennon <code>TestJob.perform_in(10.seconds)</code> suoritetaan operaatio asynkronisesti 10 sekunnin kuluttua.

Asynkroninen operaatio voi käynnistää itse itsensä, eli jos muutat koodin muotoon 

```ruby
class TestJob
  include SuckerPunch::Job

  def perform
    sleep 1
    puts "starting job..."
    sleep 10
    puts "job ready!"
    TestJob.perform_in(30.seconds)
  end
end
```

ja annat komennon <code>TestJob.perform_async</code> operaatio suoritetaan toistuvasti 30 sekunin välein niin kauan kunnes konsoli suljetaan.

> ## Tehtävä 12
>
> Nopeuta ratings-sivun toimintaa haluamasi tekniikan. Voit olettaa, että käyttäjät ovat tyytyväisiä eventual consistency -mallin mukaiseen tiedon ajantasaisuuteen.  
>
>Kirjoita ratings-kontrollerin <code>index</code>-metodiin pieni selitys nopeutusstrategiastasi jos se ei ole koodin perusteella muuten ilmeistä.
>
> Jos päädyt käyttämään asynkronisia workereita, ei koodia ole välttämättä ihan helppoa saada toimimaan täysin oikein.

## Sovelluksen koostaminen palveluista

Sovelluksen suorituskyvyn skaalaaminen onnistuu vain tiettyyn pisteeseen asti, jos sovellus on monoliittinen, kokonaan yhden tietokannan varassa, yhdellä palvelimella suoritettava kokonaisuus. Sovellusta voidaan toki optimoida ja sitä voidaan skaalata __horisontaalisesti__ eli palvelimen fyysisiä resursseja kasvattamalla.

Parempaan skaalautuvuuteen päästään kuitenkin __vertikaalisella__ skaalautuvuudella, eli sen sijaan että palvelimen fyysisiä resursseja yritettäisiin kasvattaa, otetaankin sovelluksen käyttöön useita palvelimia, jotka suorittavat sovelluksen toimintoja rinnakkain. Vertikaalinen skaalaaminen ei välttämättä onnistu triviaalisti, sovelluksen arkkitehtuuria on mukautettava. Jos sovellusta palvelee edelleen ainoastaan yksi tietokanta, voi siitä tulla pullonkaula vertikaalisesta skaalaamisesta huolimatta, erityisesti jos kyseessä on relaatiotietokanta, joiden hajauttaminen ja näin ollen vertikaalinen skaalaaminen ei ole helppoa.

Sovelluksen skaalaaminen (ja joissain tapauksissa myös sen ylläpitäminen ja laajentaminen) on helpompaa, jos sovellus on koostettu useammista erillisistä itsenäisenä toimivista keskenään esim. HTTP-protokollan välityksellä kommunikoivista __palveluista__. Sovelluksemme itseasiassa hyödyntää jo toista palvelua eli BeermappingAPI:a. Vastaavasti sovelluksen toiminnallisuutta voitaisiin laajentaa integroimalla siihen uusia palveluja.

Jos haluaisimme esim. että sovelluksemme tekisi käyttäjälle suosikkioluttyyleihin ja sijaintiin (joka saadaan selvitettyä esim. käyttäjän tekemien HTTP-kutsujen IP-osotteen perusteella, ks http://www.iplocation.net/) perustuvia ruokareseptisuosituksia, kannattaisi suosittelijasta tehdä kokonaan oma palvelunsa. Sovelluksemme keskustelisi sitten palvelun kanssa HTTP-protokollaa käyttäen.

Jos haluaisimme vastaavasti, että sovelluksemme näyttäisi käyttäjälle olutsuosituksia käyttäjän oman suosikkityylin perusteella, olisi tämän toiminnallisuuden eriyttäminen omaksi, erillisellä palvelimella toimivaksi palveluksi hieman haastavampaa, sillä suositukset todennäköisesti riippuisivat muiden ihmisten tekemistä reittauksista ja tähän tietoon käsille pääsy taas edellyttäisi olutsuosittelijalta pääsyä sovelluksemme tietokantaan. Eli jos oluiden suosittelija haluttaisiin toteuttaa omana erillisenä palvelunaan, olisi sovelluksemme rakennetta kenties mietittävä kokonaan uudelleen, jotta tieto reittauksista saataisiin jaettua ratebeer-sovelluksen ja olutsuosituspalvelun kesken.

Viime aikoina onkin noussut suosioon tyyli, jossa sovellus koostetaan [mikropalveluista (engl. micro service)](https://martinfowler.com/articles/microservices.html) eli melko pienistä, yhden erillisen tehtävän itsenäisesti hoitavista palveluista. 

## NoSQL-tietokannat

Relaatiotietokannat ovat dominoineet tiedon tallennusta jo vuosikymmenten ajan. Viime aikoina on kuitenkin alkanut jälleen tapahtumaan tietokantarintamalla, ja kattotermin [NoSQL](https://en.wikipedia.org/wiki/NoSQL) alla kulkevat "ei relaatiotietokannat" ovat alkaneet nostaa suosiotaan.

Yhtenä motivaationa NoSQL-tietokannoilla on ollut se, että relaatiotietokantoja on vaikea skaalata massivisten internetsovellusten vaatimaan suorituskykyyn. Toisaalta myös tiettyjen NoSQL-tietokantojen skeemattomuus tarjoaa sovellukselle joustavuutta verrattuna SQL-tietokantojen tarkastimääriteltyihin tietokantaskeemoihin.

NoSQL-tietokantoja on useita, keskenään aivan erilaisilla toimintaperiaatteilla toimivia, mm.
* avain/arvotietokannat (key-value databases)
* dokumenttitietokannat (document databases)
* saraketietokannat (columnar databases)
* verkkotietokannat (graph databases)

Jo meille tutuksi tullut <code>Rails.cache</code> on oikeastaan yksinkertainen avain-arvotietokanta, joka mahdollistaa mielivaltaisten olioiden tallettamisen avaimeen perustuen. Tietokannasta haku on rajoittunut hakuun avaimien perusteella ja tietokanta ei tue kannassa olevien olioiden välisiä liitoksia ollenkaan.

Uusien tietokantatyyppien noususta huolimatta relaatiotietokannat tulevat kuitenkin säilymään ja on todennäköistä että isommissa sovelluksissa on käytössä rinnakkain erilaisia tietokantoja, ja kuhunkin talletustarkoitukseen pyritään valitsemaan tilanteeseen parhaiten sopiva tietokantatyyppi, ks.
http://www.martinfowler.com/bliki/PolyglotPersistence.html

## Single sign on

Monilla sivustoilla on viime aikoina yleistynyt käytäntö, jossa mahdollistetaan sivulle kirjautuminen esim. Google-,  Facebook- tai GitHub-tunnuksilla. Sivustot siis ovat ulkoistaneet käyttäjänhallinnan ja autentikoinnin erillisille palveluille.

Autentikointi tapahtuu OAuth2-standardia (ks. https://tools.ietf.org/html/draft-ietf-oauth-v2-31) hyödyntäen, OAuth-autentikoinnin perusteista enemmän esim. osoitteessa http://aaronparecki.com/articles/2012/07/29/1/oauth2-simplified

OAuth-pohjainen autentikaatio onnistuu Railsilla helposti Omniauth-gemien avulla, ks. http://www.omniauth.org/ Jokaista palveluntarjoajaa kohti on nykyään olemassa oma geminsä, esim. [omniauth-github](https://github.com/intridea/omniauth-github)

> ## Tehtävä 13
>
> Lisää sovellukseen mahdollisuus käyttää sitä GitHub-tunnuksilla. Etene seuraavasti:
> * Kirjaudu GitHubiin ja mene [setting-sivulle](https://github.com/settings/profile). Valitse vasemmalta _developer settings_ ja sen alta _oauth apps_. Klikkaa _New OAuth app_, määrittele _homepage urliksi_ http://localhost:3000 ja _authorization callback urliksi_ http://localhost:3000/auth/github/callback
> * asenna [omniauth-github](https://github.com/intridea/omniauth-github) gem
> * Luo hakemistoon _config/initializers_ tiedosto _omniauth.rb_, jolla on seuraava sisältö:
> ```ruby
> Rails.application.config.middleware.use OmniAuth::Builder do
>  provider :github, ENV['GITHUB_KEY'], ENV['GITHUB_SECRET']
> end
> ```
> * aseta GitHubiin luomasi sovelluksen sivulla olevat _client id_ ja _client secret_ edellä määriteltyjen [ympäristömuuttujien](https://github.com/mluukkai/WebPalvelinohjelmointi2018/blob/master/web/viikko5.md#sovelluskohtaisen-datan-tallentaminen) arvoksi
> * lisää tiedostoon _routes.rb_ reitti
> ```ruby
>   get 'auth/:provider/callback', to: 'sessions#create_oauth'
> ```
> * luo reitin määrittelemä kontrollerimetodi _sessiokontrolleriin_
> * tee sovellukseen nappi, jota klikkaamalla käyttäjä voi kirjautua sovellukseen GitHub-tunnuksilla. Napin pathi on _auth/github_
> * kun kirjaudut sovellukseesi GitHub-tunnuksilla, uudelleenohjautuu selain osoitteeseen _auth/github/callback_ eli routes.rb:n määrittelyn ansioista suoritus siirtyy sessiokontrollerin metodille _create_oauth_, pääset siellä käsiksi tarvittaviin tietoihin muuttujan <code>request.env["omniauth.auth"]</code> avulla:
> ```ruby
> (byebug) request.env['omniauth.auth'].info
> #<OmniAuth::AuthHash::InfoHash email="mluukkai@iki.fi" image="https://avatars1.githubusercontent.com/u/523235?v=4" name="Matti Luukkainen" nickname="mluukkai" urls=#<OmniAuth::AuthHash Blog="" GitHub="https://github.com/mluukkai">>
> ```
> * tee sovellukset tarvittavat muutokset
> * kun sovellus toimii paikallisesti, vaihda GitHub-sovelluksen _homepage url_ ja _authorization callback url_ vastaamaan Herokussa olevan sovelluksesi urleja
>
> Muutokset eivät ole täysin suoraviivaisia:
> * sessiokontrollerin uuteen metodiin tulee kirjoittaa koodi, joka tarkastaa käyttäjän identiteetin ja luo tarvittaessa GitHub-käyttäjää vastaavan <code>User</code>-olion
> * joudut muokkaamaan <code>User</code>-modelia siten, että sen avulla hoidetaan sekä järjestelmän omaa salasanaa hyödyntävät käyttäjät, että GitHubin kautta kirjautuvat
> * tällä hetkellä <code>User</code>-olioiden validoinnissa vaaditaan, että olioilla on vähintään 4 merkin mittainen salasana. Joudut tekemään validoinnin ehdolliseksi, siten ettei sitä vaadita GitHubin tunnuksilla kirjautuvalta käyttäjältä (katso apua googlella) tai toinen vaihtoehto on generoida myös GitHubin kautta kirjautuville esim. satunnainen salasana

## Refaktorointi: luokka metodit 

Viikon 6 tehtävässä [6-7](https://github.com/mluukkai/WebPalvelinohjelmointi2018/blob/master/web/viikko6.md#teht%C3%A4v%C3%A4-6-7-kahden-teht%C3%A4v%C3%A4n-arvoinen) kehoitettiin tekemään luokille _Beer_, _Brewery_ ja _Style_ luokkametodit, joiden avulla kontrollerin on helppo selvittää saa reittausten perusteella parhaat panimot, oluet ja oluttyylit.

Metodit ovat kaikissa luokissa täsmälleen samat:

```
class Beer < ApplicationRecord
  # ...

  def self.top(how_many)
    sorted_by_rating_in_desc_order = all.sort_by{ |b| -(b.average_rating || 0) }
    sorted_by_rating_in_desc_order[0, how_many]
  end
end
```

Viikolla 2 siirrettiin luokkien määrittelemiä samanlaisia _oliometodeita_ [yhteiseen moduuliin](https://github.com/mluukkai/WebPalvelinohjelmointi2018/blob/master/web/viikko2.md#yhteisen-koodin-siirto-moduuliin). 

Myös luokkametodeja voidaan siirtää yhteiseen moduuliin, tekniikka ei kuitenkaan ole täysin sama kuin oliometodeja käytettävissä

> ## Tehtävä 14
>
> Refaktoroi koodisi siten, että luokkien _Beer_, _Brewery_ ja _Style_ metodi _def self.top(how_many)_ määritellään moduulissa. Saat vihjeitä toteutukseen esim. googlaamalla _ruby module static method_ 


> ## Tehtävä 15
>
> Kurssi on tehtävien osalta ohi ja on aika antaa kurssipalaute osoitteessa https://ilmo.cs.helsinki.fi/kurssit/servlet/Valinta

## Tehtävien palautus

Commitoi kaikki tekemäsi muutokset ja pushaa koodi Githubiin. Deployaa myös uusin versio Herokuun. Muista myös testata rubocopilla, että koodisi noudattaa edelleen määriteltyjä tyylisääntöjä. 

Tehtävät kirjataan palautetuksi osoitteeseen https://studies.cs.helsinki.fi/courses/#/rails2018

## Mitä seuraavaksi?

Jos Rails kiinnostaa, kannattaa tutustumista jatkaa esim. seuraaviin suuntiin

* https://leanpub.com/tr5w The Kirja. Kannattaa __ehdottomasti__ hankkia ja lukea kannesta kanteen
* http://guides.rubyonrails.org/ Paljon hyvää asiaa...
* http://railscasts.com/ erinomaisia yhteen teemaan keskittyviä videoita. Uusia videoita ei valitettavasti ole tullut yli vuoteen, toivottavasti sivu aktivoituu vielä. Useimmat maksulliset pro-episodit näköjään löytyvät youtubesta...
* https://www.ruby-toolbox.com/ apua gemien etsimiseen
* [Eloquent Ruby](http://www.amazon.com/Eloquent-Ruby-Addison-Wesley-Professional-Series/dp/0321584104) erinomainen kirja Rubystä.

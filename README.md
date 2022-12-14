# API Weather module

 Tässä projektissa luodaan ja asennetaan moduuli, joka näyttää US National Weather Service API:n avulla tämän hetkisen lämpötilan.
 
 
 
 # Licence
 
  GNU General Public License v3.0.
 

 ![SaltProject_altlogo_teal](https://user-images.githubusercontent.com/117189365/207580235-76861f6b-69bd-4cfc-8445-894daa043de6.png)

 
 
 # Getting started


 Ennenkuin pääsemme alkuun tulee olla Salt master ja vähintään yksi minion asennettuna ja conffattuna. Sen lisäksi tarvitaan python-pip.
 
 1. Aloitetaan luomalla /srv/salt/ hakemisto, jos sitä ei vielä ole. Sinne sijoitetaan kaikki päätiedostot sekä salt-state tiedosto.
  `mkdir /srv/salt/`
  
 2. Luodaan top.sls tiedosto /srv/salt/ hakemistoon, joka on tässä tapauksessa pääkonfigurointitiedosto. Lisätään sinne:
```
base:
  '*':
    - weather
```
3. Seuraavaksi luodaan state tiedosto, joka nimetään weather.sls ja sinne lisätään seuraava:
```
python-pip:
  pkg.installed

requests:
  pip.installed:
    - require:
      - pkg: python-pip
```
Tämän jälkeen otetaan muutokset käyttöön komennolla
`salt '*' state.apply`
Mikäli kaikki toimii niinkuin pitääkin, niin luodaan uusi hakemisto /srv/salt/_modules Jonne lisätään moduuli weather.py
Sinne lisätään koodin pätkä:

```
import logging
try:
    import requests
    HAS_REQUESTS = True
except ImportError:
    HAS_REQUESTS = False

log = logging.getLogger(__name__)

__virtual_name__ = 'weather'

def __virtual__():
    '''
    Only load weather if requests is available
    '''
    if HAS_REQUESTS:
        return __virtual_name__
    else:
        return False, 'The weather module cannot be loaded: requests package unavailable.'


def get(signs=None):
    '''
    Gets the Current Weather

    CLI Example::

        salt minion weather.get KPHL

    This module also accepts multiple values in a comma seperated list::

        salt minion weather.get KPHL,KACY
    '''
    log.debug(signs)
    return_value = {}
    signs = signs.split(',')
    for sign in signs:
        return_value[sign] = _make_request(sign)
    return return_value

def _make_request(sign):
    '''
    The function that makes the request for weather data from the National Weather Service.
    '''
    request = requests.get('https://api.weather.gov/stations/{}/observations/current'.format(sign))
    conditions = {
        "description:": request.json()["properties"]["textDescription"],
        "temperature": round(request.json()["properties"]["temperature"]["value"], 1)
    }
    return conditions
```
## Excecuting and testing if module works as it should.

1. Jotta saadaan kutsu minioneilta ajetaan komento:
 `salt '*' saltutil.sync_modules`
 
2. Seuraavaksi ajetaan ja testataan moduuli salt masterilla.
`salt '*' weather.get KPHL`
Nyt sen pitäisi tulostaa ruudulle sää sekä lämpötila.
    
  

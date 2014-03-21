# Tests<br/>Qualimétrie

<!-- .slide: data-background="zenika/images/title-background.png" -->



## Plan

<!-- .slide: class="toc" -->

- [Build et génération du livrable](#/1)
- [Gestion des dépendances](#/2)
- **[Tests et qualimétrie](#/3)**
- [Productivité](#/4)
- [Intégration continue](#/5)
- [Debugging et optimisation](#/6)



## Objectifs

- Feedback rapide sur la fiabilité du code à tous les niveaux
  - Tests unitaires
  - Tests de bout-en-bout
  - Analyse statique
  - Calcul de la couverture



## Analyse statique

<figure>
    <img 
      src="assets/images/jshint-logo.png" 
      alt="JSHint logo"  
      width="50%"
      style="margin-top: 20%"/>
</figure>



## JSHint

- http://jshint.com
  - Analyse possible directement sur le site
- Fork de [JSLint](http://www.jslint.com/) de Douglas Crockford
- Recherche les erreurs
- Fait des remarques de styles
- Complètement configurable
- Installable avec NPM
  - `npm install -g jshint`
  - `jshint <fichier.js>`



## Exemple

```javascript
function une_accolade_par_là() {
    une_autre_variable = 2 * variable
    var variable = 0
    la_même = '' || [];
    if (variable == la_même) return undefined
    consle.log("que dit jshint ?")
    return 1
}

function une_accolade_par_ci()
{
    return
    {
        un: 'objet,'
        bien: 'rempli'
    }
}

console.log(une_accolade_par_ci())
console.log(une_accolade_par_là())
```



## Exemple

- Ce code s'exécute sans erreur : il affiche `undefined` deux fois
- Sans analyse ou tests, il pourrait partir en production !
- Qu'en dit JSHint ? 22 erreurs !
  - `Missing semicolon` : oubli du point-virgule
  - `'une autre variable' is not defined` : oubli du `var`
  - `'consle' is not defined` : faute de frappe
  - `Line breaking error 'return'` : retour à la ligne intempestif
  - `Expected an assignment or function call and instead saw an expression` :
  syntaxe invalide, ici il manque une virgule



## Intégration avec Grunt

- Plugin `grunt-contrib-jshint`

```javascript
jshint: {

  options: {
    curly: true, // forcer les accolades
    eqeqeq: true, // forcer ===
    /* ... */
  },

  src: ['src/**/*.js'],
  test: ['test/**/*.js'],
},
```



## Intégration aux éditeurs de texte

- Plugin pour de nombreux éditeurs
  - Notepad++, Gedit
  - Sublime Text, TextMate
  - Vim, Emacs
  - Visual Studio, Eclipse
  - IntelliJ, WebStorm (intégré en standard)



## Tests unitaires

<figure>
    <img src="assets/images/jasmine-logo.png" alt="Jasmine logo"  width="50%"/>
    <figcaption>Behavior-Driven Javascript</figcaption>
</figure>



## Jasmine

- http://jasmine.github.io/2.0

```javascript
describe('a parrot', function() {

  var sut = parrot();
  var message = 'hello!';

  it("repeats what it's told", function () {
    sut.onTold(message);
    expect(sut.repeat()).toBe(message);
  });
});
```

- Divers matchers : `toBeEqual`, `toContain`, `toBeLessThan`,
`toBeTruthy`... + matchers custom



## Setup & Teardown

```javascript
describe('a parrot', function() {

  var sut = parrot();
  var message = 'hello!';

  beforeEach(function() { 
    sut.onPet(); 
  });

  afterEach(function() { 
    sut.onFed(); 
  });

  it("repeats what it's told", function () {
    sut.onTold(message);
    expect(sut.repeat()).toBe(message);
  });

});
```



## Mocks / Spies

```javascript
describe('a parrot', function() {

  var sut = parrot();
  var message = 'hello!';

  beforeEach(function() { 
    spyOn(sut, 'onTold'); 
  });

  it("can be spied on", function () {
    sut.onTold(message);
    expect(sut.onTold).toHaveBeenCalledWith(message);
  });

});
```



## Lancer les tests

- Dans un navigateur
  - Ecrire une page HTML qui importe Jasmine, le code à tester, les tests
  - Ouvrir la page dans le navigateur de référence
  - Une telle page est fourni avec Jasmine, il faut simplement modifier les 
  `script[src]`
- Dans Node, à l'aide du projet `jasmine-node` (Jasmine 1.3)
  - `npm install -g jasmine-node`
  - `jasmine-node <fichiers/dossiers de tests>`



## Alternatives

- [Mocha](http://visionmedia.github.io/mocha/)
  - API très proche de Jasmine
  - Conçu pour Node mais supporte les navigateurs
  - Plus flexible mais plus difficile à appréhender (pas d'API d'assert ni de
  mock embarquées)
- [QUnit](https://qunitjs.com/)
  - API standard xUnit
  - Conçu pour les navigateurs, peut fonctionner sous Node avec à l'aide de
  projets tierce-partie



## Automatisation des tests unitaires

<figure>
    <img 
      src="assets/images/karma-logo.png" 
      alt="Karma logo"  
      width="60%"
      style="margin-top: 20%"/>
    <figcaption>Spectacular Test Runner for Javascript</figcaption>
</figure>



## Karma

- Créé par l'équipe AngularJS
- Tourne sur Node
- Il exécute atuomatiquement les tests
  - dans plusieurs navigateurs
  - à chaque modification du code
- Indépendant du framework de test
  - Compatible Jasmine, Mocha, QUnit et autres



## Installation
- `npm install -g karma-cli`
- `npm install karma` + les plugins voulus
  - `karma-jasmine`
  - `karma-firefox-launcher`
  - ...
- `karma init` crée un fichier de configuration `karma.conf.js` interactivement
- `karma start` lance Karma en continue
  - ajouter l'option `--single-run` pour passer les tests une fois



## Exemple de karma.conf.js

```javascript
module.exports = function(config) {
  config.set({
    frameworks: ['jasmine'], 

    files: [ // Inclus le code à tester
      'src/*.js',
      'test/*.js',
    ],

    browsers: ['Chrome', 'Firefox'],

    // Relancer les tests à chaque modification d'un fichier
    autoWatch: true, 

    // Une seule passe de test
    singleRun: false,
  });
};
```



## Couverture de test

- `npm install karma-coverage`

```javascript
module.exports = function(config) {
  config.set({

    /* ... */

    preprocessors: {
      "*.js": ['coverage'],
    },

    reporters: ['coverage'],

  });
};
```



## Couverture de test

<img src="assets/images/coverage-example.png" alt="Test Coverage Example" 
width="90%" class="with-border"/>



## Tests bout-en-bout

<figure>
    <img src="assets/images/angular-logo.png" alt="Angular logo"  width="45%"/>
    <figcaption>Protractor: E2E test framework for Angular apps</figcaption>
</figure>



## Protractor

- Créé par l'équipe AngularJS
- Tourne sur Node
- Basé sur Selenium
  - Tests par automation du navigateur
  - Nécessite un serveur Selenium
  - Reprend le style de l'API Selenium en ajoutant des spécificités Angular
- API pour les tests : Jasmine ou Mocha



## Mise en route

- `npm install -g protractor`
- `webdriver-manager update` + `webdriver-manager start` pour installer et
lancer un serveur Selenium
- `protractor protractor.conf.js`

```javascript
exports.config = {
  seleniumAddress: 'http://localhost:4444/wd/hub',
  capabilities: {
    'browserName': 'firefox'
  },
  specs: ['spec.js'],
};
```



## Exemple de test

```javascript
describe('angularjs homepage', function() {

  it('should greet the named user', function() {
    browser.get('http://www.angularjs.org');

    // Cherche les input avec ng-model=yourName
    element(by.model('yourName')).sendKeys('Zenika');

    // Cherche les éléments bindés à yourName
    // Exemple : <h1>Hello \{{yourName}}</h1>
    var greeting = element(by.binding('yourName'));

    expect(greeting.getText()).toEqual('Hello Zenika!');
  });

});
```



<!-- .slide: data-background="zenika/images/questions.png" -->
<!-- .slide: data-background-size="30%" -->
# Cours sur Symfony

### Créer des objets qui servent de données d'essai dans un fichier fixtures ###

ExempleFixture.php : on va en créer plusieurs à l'aide d'une boucle for

>```  {
>    public function load(ObjectManager $manager)
>    {
>        for ($i = 0; $i < 3; $i++) {
>            $exempleFixture = new Exemple();
>                ->setLuckyNumber($i)
>                ->setDescription("'Je suis le flocon {$i}")
>                ->setCreatedAt(new \DateTime('now'));
>
>            $manager->persist($exempleFixture);
>        }
>        $manager->flush();
>    }
> }
```


**Envoyer les fixtures dans la base de données**
>Dans la console écrire :
> symfony doctrine:fixtures:load --append

>--append n'est utile que si on ne souhaite pas écraser les données déjà dans la base de données.


**Créer une nouvelle fonction et une route pour afficher les données dans une page grâce à un controller**



```
/**
 * @Route("/snowflake", name="app_snowstorm")
 */
public function snowtime(SnowflakeRepository $snowflakeRepository): Response
{
  $snowflakes = $snowflakeRepository->findAll();
  return $this->render('snowstorm/index.html.twig',
            compact('snowflakes')
        );
    }
```

**Afficher les données dans la page index.html.twig ou dans une autre page**

```
{% extends 'base.html.twig' %}

{% block title %}Symfony 2021 blog !{% endblock %}


{% block body %}
    <section class="container my-3">

        <h1>Welcome on our new blog !</h1>
        <p><small>It's made with Symfony 5</small></p>


        {% for exemple in exemples %}

            <div class="card my-2 p-2">

            </div>

        {% endfor %}

    </section>

{% endblock %}

```

**Créer des routes pour afficher une donnée par Id ou slug**
Se rendre dans le fichier Controller

>methode 1 (fastidieuse): On récupère grâce à l'Id et on spécifie que l'on veut un digit grâce à l'option requirements.

```
/**
   * @Route("/snowflake/{id}", name="app_snowstorm_show", requirements={"id"="\d+"})
   */
  public function snowtimeById(SnowflakeRepository $snowflakeRepository, $id): Response
  {
      $snowflakeById = $snowflakeRepository->findOneById($id);
      return $this->render('snowstorm/details.html.twig', [
          'snowflakeById' => $snowflakeById
      ]);
  }
```
>methode 2 (automatisée par Symfony): On passe directement notre entité $snowflake et il trouve directement son id en faisant -getId() qui est une methode intégrée. Cela fonctionne uniquement pour l'Id et la chaîne de caractères appelée slug 'slug' . Il faut que l'id ou le get soient uniques. Ici aussi on spécifie que l'on veut un digit grâce à l'option requirements.
```
 /**
   * @Route("/snowflake/{id}", name="app_snowflake_show", requirements={"id"="\d+"})
   */
  public function snowtimeById(Snowflake $snowflake): Response
  {
      return $this->render('snowstorm/details.html.twig', [
          'snowflake' => $snowflake
      ]);
  }
```
>Ensuite il faut créer un nouveau template par exemple details.html.twig pour afficher les données et lui assigner la variable $snowflake qui est la réponse à la fonction snowtimeById();


-----------------------------------------------------------------------

## Créer un formulaire ##

### Créer une nouvelle route et sa fonction dans le fichier Controller: ###

>On demande à notre page New de créer un formulaire.
> Ressource :
> [Créer un blog avec Symfony : les formulaires kaherecode](https://www.kaherecode.com/tutorial/creer-un-blog-avec-symfony-les-formulaires)  
> [Créer un formulaire avec Symfony nouvelles techno](https://nouvelle-techno.fr/actualites/symfony-4-creer-un-blog-pas-a-pas-utiliser-les-formulaires)


**Dans la console :**
> symfony console make:form NomDeMonEntitéType

>Il  suffit de suivre les instructions. Et un nouveau dossier formulaire est construit.

**Dans l nouveau dossier Form, il y a le fichier RegistratioFormType.php**
>On trouve la fonction buildForm().

- Si on veut, on peut ajouter dans ce builder les type pour chaque champs mais il le fait automatiquement.

- Si on veut ajouter automatiquement un bouton envoyer dans le formulaire il faut ajouter cette ligne dans le buildForm().
``$builder->add('Envoyer', SubmitType::class);``

**Dans la partie template**

>methode 1 :

```
{% extends "base.html.twig" %}

{% block title %}Ajouter un flocon
{% endblock %}

{% block body %}
	<div class="container">
		<div class="row">
			<div class="col s12 m12 l12">
				<h4>Ajout d'un article</h4>
			</div>
		</div>
		<section>

			{{form(form)}}

    </section>
	</div>
{% endblock %}

```

>methode 2

```
{% extends "base.html.twig" %}

{% block title %}Ajouter un flocon
{% endblock %}

{% block body %}
	<div class="container">
		<div class="row">
			<div class="col s12 m12 l12">
				<h4>Ajout d'un article</h4>
			</div>
		</div>
		<section>

    	{{form_start(form)}}
			{{form_errors(form)}}
			{{form_widget(form)}}
			{# <input type="submit" class="btn btn-primary"> #}
			{{form_end(form)}}

		</section>
	</div>
{% endblock %}
```
>methode 3

```
{% extends "base.html.twig" %}

{% block title %}Ajouter un flocon
{% endblock %}

{% block body %}
	<div class="container">
		<div class="row">
			<div class="col s12 m12 l12">
				<h4>Ajout d'un article</h4>
			</div>
		</div>
		<section>

    {{form_start(form)}}

      {{form_errors(form)}}
      <div class="form-row form">
          <div class="form-label">
            {{form_label(form.name)}}
          </div>
          <div>
            {{form_widget(form.name)}}
          </div>
        </div>

        <div class="form-col">
          {{form_label(form.description)}}
          {{form_row(form.description)}}
        </div>

        <div class="form-col">
          {{form_label(form.luckyNumber)}}

        <div class="">
          {{form_widget(form.luckyNumber)}}
        </div>
      </div>

<input type="submit" class="btn btn-primary my-2" value="{{submitButtonText}}"/>
{{form_end(form)}}

		</section>
	</div>
{% endblock %}
```

**Dans le Controller:**
> On indique à la page qu'elle attend une requête de type GET/POST en lui précisant dans la route. On ajoute donc en paramètre de la fonction, la requête qu'elle reçoit. Afin de créer notre entité et de l'envoyer dans la base de donnée, on utilisera aussi $manager qui est de la classe EntityManagerInterface et qui gère toutes les entités.
```
/**
 * @Route("/new", name="app_new", methods="GET|POST")
 */
public function new(Request $request, EntityManagerInterface $manager): Response
{
    $snowflake = new Snowflake();
    $form = $this->createForm(SnowflakeType::class, $snowflake);
    $form->handleRequest($request);
    if ($form->isSubmitted() && $form->isValid()) {
        $manager->persist($snowflake);
        $manager->flush();
        return $this->redirectToRoute('app_snowflake');
    }
    return $this->render(
        'snowstorm/create.html.twig',
        ['form' => $form->createView()]
    );
}
```

> Dans la logique, on créé un nouvel objet $snowflake. On créé aussi un formulaire $form qui est de de la classe SnowflakeType et  

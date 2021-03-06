AclTree Bundle
==============
AclTree Bundle allows you to create hierarchy relationship between your entities for ACL Permissions.

For example, If I have `Edit` permissions for `Dan Brown books` entity, I will able to edit also the author `Dan Brown`, and all of his books.

Table of contents:
------------------
- [Introduction](#acltree-bundle)
- [Installation](#installation)
- [Usage](#usage)
    - [Defining entity parent](#defining-entity-parent)
    - [Using the voter](#using-the-voter)
    - [AclTreeHelper](#using-the-acltree-helper)
    - [AclUsersHelper](#using-the-acltree-helper)
- [Changing the default MaskBuilder](#changing-the-default-maskbuilder)


Installation:
-------------
Add the following line into your `composer.json`, at the `require` section:

##### composer.json
```json
    "require": {
       "godisco/acltree-bundle": "dev-master"
```

Add the **AclTree Bundle** to your `AppKernel` (at the `registerBundles()` section):

##### app/AppKernel.php
```php
class AppKernel extends Kernel
{
    public function registerBundles()
    {
        $bundles = array(
            // ...
            new GoDisco\AclTreeBundle\AclTreeBundle(),
    }
```


Usage:
------
Using the AclTree is pretty simple!
First, you need to [Define your entities parents](#defining-entity-parent), then just use the voter to know if access is granted to user, or apply the helpers for complex queries.

- **[Defining entity parent](#defining-entity-parent)**
- **[Using the voter](#using-the-voter)**
- **[AclTreeHelper](#using-the-acltree-helper)** - Filtering entities by ACL.
- **[AclUsersHelper](#using-the-acltree-helper)** - Showing all the users who have access by ACL.

<br />

### Defining entity parent
Just add the `@AclParent` annotation to the parent member of your entity.

#### Example:
```php
use GoDisco\AclTreeBundle\Annotation\AclParent;
use Acme\AuthorBundle\Entity\Author;

/**
 * Book
 * @ORM\Table
 * @ORM\Entity
 */
class Book
{
    /**
     * @var Author
     *
     * @ORM\ManyToOne(targetEntity="Acme\AuthorBundle\Entity\Author")
     * @ORM\JoinColumn(name="author_id", referencedColumnName="id")
     * @AclParent
     */
    private $author;
}
```

*** Don't forget to include the annotation by adding the `use GoDisco\AclTreeBundle\Annotation\AclParent;` part! ***

### Using the voter
You can use the regular [Symfony ACL voter](http://symfony.com/doc/current/cookbook/security/acl.html#checking-access), in order to know if access is granted to the user:
```php
$vote= $this->get('security.context')->isGranted('VIEW', $entity);
```
(*) For more Information about using voters, visit the [Symfony documentation](http://symfony.com/doc/current/cookbook/security/voters_data_permission.html).

<br />

In addition, you can also use the voter as annotation (duh?):
```php
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Security;

class BookController extends Controller
{
    /**
     * @Security("is_granted('VIEW', post)")
     */
    public function showAction(Post $post)
    {
        //...
    }
}
```
(*) For more Information about using security annotations, visit the [Symfony documentation](http://symfony.com/doc/current/bundles/SensioFrameworkExtraBundle/annotations/security.html#usage).


### Using the AclTree helper
For filtering only entities the user have access to, apply the **@acl.tree.helper** service on your `QueryBuilder` object:

#### Methods:
- **apply(QueryBuilder $queryBuilder, [array $permissions = array("VIEW"), UserInterface $user = null])**
    - ***$queryBuilder***(*) - `QueryBuilder` object which select all the entities, **before the filtering**.
    - ***$permissions*** - array of permissions to check, default will check only for `VIEW`
    - ***$user*** - User entity to check, default will check for the logged-in user
    - ***returns*** modified **`Query`** object.

#### Example:
```php
    /** @var \Doctrine\ORM\EntityManager\EntityManager $em */
    $em = $this->getDoctrine()->getManager();
    /** @var \GoDisco\AclTreeBundle\Security\Helper\AclTreeHelper $aclHelper */
    $aclHelper = $this->get("acl.tree.helper");
    
    $qb = $em->createQueryBuilder();
    $qb = $qb->select('e')
        ->from('GoDisco\EventBundle\Entity\Event', 'e')
        ->join("e.line", "l");

    // The query object returned here is a clone obj so, you can always use $qb->getQuery() to get the original query obj
    $query = $aclHelper->apply($qb);  // showing for current logged-in user
    $result = $query->getArrayResult();
    
    return $result;
```

### Using the AclUsers helper
For showing all the users who have directly access to the entity, apply the `acl.object.users` service on your `Entity` object:

#### Methods:
- **get($entity, $user_class[array $permissions = array("VIEW"))**
    - ***$entity***(*) - entity to check
    - ***$user_class***(*) - the class of the user object
    - ***$permissions*** - array of permissions to check, default will check only for `EDIT`
    - *returns* list of users.
    
#### Example:
```php
    /** @var \Doctrine\ORM\EntityManager\EntityManager $em */
    $em = $this->getDoctrine()->getManager();

    $repo = $em->getRepository("VenueBundle:Venue");
    $entity = $repo->find(1);

    /** @var \GoDisco\AclTreeBundle\Security\Helper\AclUsersHelper $aclHelper */
    $aclHelper = $this->get("acl.object.users");
    
    $users = $aclUsers->get($entity, 'Acme\UserBundle\Entity\User'); // showing for current logged-in user
    return $users;
```


Changing the default MaskBuilder
---------------------------------
In some cases, you may wish to change the default `MaskBuilder`, to a custom mask map.
You can just modify the MaskBuilder by overriding the `security.acl.mask_builder` parameter.

For instance, example for changing the AclTree to use The Sonata's ACL MaskBuilder:
```yaml
parameters:
    security.acl.mask_builder: Sonata\AdminBundle\Security\Acl\Permission\MaskBuilder
```
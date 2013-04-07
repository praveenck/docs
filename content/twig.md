Twig Templates
==============

Here are the main advantages that draw us towards replacing the .phtml templates
used for earlier Magento extensions with a templating engine called Twig.

The move helps enforce a desirable isolation of the presentation layer from
domain and business logic and data, which is a fundamental goal of our new
extension approach. Current .phtml templates allow inclusion of business logic
and SQL queries within the presentation layer, which  can create serious
challenges for maintenance and upgrades. We see the following additional
benefits from separating the presentation logic:

-   Theme switching becomes much easier and more predictable, because themes are
    confined to presentation logic and won't need to be disentangled from domain
    and business logic.

-   Theme and template security issues are much reduced, for the same reason.

-   Designers can change templates with more confidence--again, for the same
    reason.

About Twig
----------

We chose Twig because integrating it into Magento will give you the following
features and benefits:

-   Twig is concise. Gone are the days of the unavoidably messy .phtml files,
    since the page html is separate from the code that populates the page. See
    the [before and after comparison](#before-after) below to check out the difference.

-   The separation of front-end concerns from implementation concerns enables
    you to hire the best front-end designers while ensuring that your PHP
    developers work on what they do best.

-   Twig autoescapes by default, ensuring that your pages will avoid
    embarrassing mistakes that can happen if you forget to wrap output
    in *htmlspecialchars*.

-   We haven't created extra work for you. Your existing PHP templates will
    continue working as expected but we hope the improvements offered by Twig
    will encourage you to switch soon.

Learn more about [Twig][2].

[2]: <http://twig.sensiolabs.org/>

Twig vs. PHTML
--------------



-   Twig has cleaner code: you can focus on what your page does without needing
    to also parse the business logic embedded in the .phtml code

-   With the separation of font-end concerns, your front-end developer can focus
    completely on the page look-and-feel.

-   Twig is integrated with IDEs. Twig files are handled cleanly by IDEs,
    ensuring that your front-end developer is able to spot mistakes in this file
    with the help of the IDE instead of repeatedly running to code to see if
    anything is wrong. This will result in more productivity.

-   Code that intuitively makes sense: compare the line “\$this-\>__..” with
    “..|translate”. Both of them do the same thing but the latter instantly
    tells you what is going on whereas the former will require a few lookups in
    code.

### <a id="before-after"></a>Before and After   

PHTML

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

```<?php if ($this->canEmailToFriend()): ?>
  <p class="email-friend">
    <a href="<?php echo 
      $this->helper('Mage_Catalog_Helper_Product')
        ->getEmailToFriendUrl($_product) ?>">
      <?php echo $this->__('Email to a Friend') ?>
    </a>
  </p>
<?php endif; ?>```

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~



Twig

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```
{% if product.canEmailToFriend %}
  <p class="email-friend">
    <a href="{{ links.emailToFriend }}">
    {{ 'Email to a Friend'|translate }}
    </a>
  </p>
{% endif %}```

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~



How to Get Started
------------------

This section describes how to begin using Twig in your existing Magento
extension or to build your new Magento extension with Twig. Our reference
implementation is in the Mage/Catalog extension.

-   Start by looking at our sample implementation in Mage/Catalog. You will see
    the changes that we made to layout.xml to use the new datasources and twig
    templates in place of the previous .phtml and model files.

-   Switching from an existing .phtml template to a twig file is as simple as
    taking the contents of the phtml file, pasting the contents to a twig file
    that you create in the same directory, and then removing the PHP code from
    this file. You end up with just html markup in your new twig file.

-   Using the original phtml file as a reference and the Service document
    available in this repo, ensure that the data needed by the phtml file is
    available in the necessary service. You might need to define a new service
    as defined in the Service IDL write-up, [here][8].

[8]: <http://praveenck.github.io/docs/service-idl/>

### Concrete Example

The Twig half of the above [before and after comparison][1] shows the contents
of `app/code/Mage/Catalog/view/frontend/product/view/options/type/date.twig`.
You might ask yourself how the various variables used in this template are made
available to it. This section will try to show you how.

[1]: <#before-after>

-   \*option\* (instance of Mage_Catalog_Model_Product_Option): This is
    available to the block class associated with this template, i.e.
    Mage_Catalog_Block_Product_View_Options_Type_Date. Any methods accessible to
    the "option" instance are callable within the template, such as
    \*isRequired\*.

-   \*createBlock / getSelectFromToHtml / viewFileUrl\*: These are defined as a
    Twig function in Mage_Core_Block_Template_Engine_TwigExtension; the latter
    is part of the template engine subsystem included with Magento 2.
    Incidentally, this template engine subsystem is responsible for ensuring
    that you can include a .phtml block as part of your twig template. This
    makes it easy to gradually convert over to using Twig.

### Warnings and Gotchas

-   Twig uses its own cache, which currently is stored in the same directory as
    the other Magento Twig caches. To manually clear, do the following: `rm -rf
    var/cache/??` while in the Magento base directory.

-   Twig's cache is controlled by the BLOCK_HTML cache setting in Magento Admin.
    Thus, disabling the "Blocks HTML Output" cache in Admin-\>Cache Management
    also disables the Twig cache. As you are most likely already aware, it is
    always best to keep Magento caches turned off during development.

-   Twig's performance is further improved if you use APC or some other PHP
    bytecache scheme.

Conclusion
----------

We hope you can see some of the flexibility and power the use of Twig will bring
to your development efforts on Magento. Our aim is to make development fun for
you again by separating front-end design from back-end work and ensuring that
you are looking at clean, easy-to-read, and easy-to-debug code.

-   [Twig Templates][3]

[3]: <twig/>

-   [Service IDL][4]

[4]: <service-idl/>

-   [Service Implementation][5]

[5]: <service-impl/>

-   [Creating Web APIs][6]

[6]: <web-api/>

-   [Extension Configuration][7]

[7]: <option-list/>

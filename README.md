# Issues with Breakdance Builder (Episode 2)

This is the second version of code and instuctions for correcting or finding workaround for some issues with the Breakdance WordPress Builder.

The first version (DEPRECATED) can be found here: https://github.com/johnosmond/breakdance-issues-01

The first version has a work around for Issue #2 but this version has a full fix.

## Issues

### Issue #1

If you build your menus with the Breakdance Menu Builder and include drop-down menus, the 'Active' settings that you implement do not apply to the items in any drop-down menu item. They apply only to top-level menu items.

As is common, use the Breakdance interface to set the 'Active' settings to the desired look. In my example in the video I just change the text color to 'red'. Do this for both the 'Desktop' and 'Mobile' menus.

### Issue #1.1

(because it's related)

The 'active' settings also do not bubble up to the parent when a submenu page is active.

To be clear, the 'active' settings apply to the top-level menu item when that page is the active page, but when any of the items in the drop-down is the active page, neither the item in the drop-down nor the parent at the top get the 'active' settings applied to them.

### Issue #2

The mobile menu does not allow the top-level item of a drop-down menu to click through to it's linked page. When in mobile view, the top-level becomes a selector only.

## Preparations

In order to solve these issues you will need a child theme. Breakdance recommends disabling the theme in the BD settings. However, if you do select to keep your theme they recommend using the 'Breakdance Zero Theme'. That is what we will do.

So, in preparation follow these steps:
* Change the theme settings in BD to 'Keep Theme'.
* Upload and activate the BD Zero Theme.
* Open your application in VS Code (or another editor of your choice).
* In the 'breakdance-zero-theme-master' folder, create a new folder called 'js'.
* In that folder create a new file called 'menu.js'.

We'll use the 'menu.js' file and the theme's 'functions.php' file in the solutions.

## Solutions

Here is how I deal with these issues.

### Issue #1

(This one's easy.)

Add this CSS to the custom CSS for the menu builder. Be sure that your viewport is set to 'Desktop' level.

```
.breakdance-dropdown-item--active .breakdance-dropdown-link__label {
  color: red;
}
```

In other words, select the 'Menu Builder' in the structure on the right. Then select the 'Advanced' tab on the left. In the 'Custom CSS' block, remove the default code, and paste the code written above.

That should make the items in the drop-down menu highlight in red (or whatever color you want).

### Issue #1.1 

(This one is a bit harder.)

Open the 'menu.js' file you created in VS Code and paste this code:

```
// change the color of the top-level link in the dropdown menu 
// if the current URL matches the URL of a submenu item
document.addEventListener('DOMContentLoaded', function() {
    // Get the current URL of the page
    const currentUrl = window.location.href;

    // Find all the dropdown menus that contain submenu items
    const dropdownMenus = document.querySelectorAll('.breakdance-dropdown');

    dropdownMenus.forEach(dropdown => {
        // Get the top-level link of the dropdown
        const topLevelLink = dropdown.querySelector('.breakdance-dropdown-toggle > a');
        
        // Get all submenu links
        const submenuLinks = dropdown.querySelectorAll('.breakdance-dropdown-link');

        // Check each submenu link to see if its href matches the current URL
        submenuLinks.forEach(link => {
            if (currentUrl.includes(link.href)) {
                // If a submenu link matches the current URL, change the top-level link color to red
                topLevelLink.style.color = 'red';
            }
        });
    });
});
```

Then open your 'functions.php' file for the Zero Theme and paste this at the bottom:

```
// added script for menu customization
function my_custom_scripts() {
    wp_enqueue_script('my-custom-js', get_template_directory_uri() . '/js/menu.js', array(), false, true);
}
add_action('wp_enqueue_scripts', 'my_custom_scripts');
```

Refresh your page and you should see that the parent of the active submenu page highlights in red.

## Issue #2

To fix Issue #2 we add some more JavaScript to the menu.js file. This will override the default behavior programmed in by Breakdance.

Just add this code to menu.js:

~~~
// Prevent the default behavior of the top-level link in the dropdown menu
// and redirect to the URL of the submenu item 
// if the window width is less than or equal to a certain breakpoint
document.addEventListener('DOMContentLoaded', function() {
    var menuLinks = document.querySelectorAll('.breakdance .breakdance-menu-list > li a.breakdance-menu-link');

    // Check if the menu links are correctly selected, log to console
    console.log(menuLinks);

    function handleMenuLinkClick(event) {
        var windowWidth = window.innerWidth;
        var mobileBreakpoint = 480; // Adjust if your mobile breakpoint differs

        if (windowWidth <= mobileBreakpoint) {
            // Check if mobile behavior needs to be different
            if (this.parentElement.classList.contains('breakdance-dropdown-toggle')) {
                event.preventDefault(); // Prevent opening submenu
                window.location.href = this.href; // Redirect to the link
            }
        }
    }

    menuLinks.forEach(function(link) {
        link.addEventListener('click', handleMenuLinkClick);
    });
});
~~~

## Conclusion

I believe that is all. I hope that helps you out.

You can find a video about this here: 

-- John Osmond

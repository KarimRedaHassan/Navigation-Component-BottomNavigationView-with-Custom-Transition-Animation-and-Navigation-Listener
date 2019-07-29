# Navigation-Component-BottomNavigationView-with-Custom-Transition-Animation-and-Custom-Navigation-Listener
This Tutorial will discuss how to implement custom transition animation when using Navigation Component provided by Android Jetpack and BottomNavigationView.

# Prerequisite:
This tutorial assumes that you are familiar with the following:
1. Android Jetpack Navigation Component
2. BottomNavigationView

# Discussion
If you are familiar with Android Jetpack Navigation Component, you will find that it's too easy to combine your navigation graph with a BottomNavigationView. You will only use one line of code to make this done

        NavigationUI.setupWithNavController(bottomNav, navController);

As you can see here, there is no much control for you over the way of communication between the NavController and BottomNavigationView. The NavigationUI class will handle everything for you starting with making the navigation when you press on a menu item, applying default animation for transition, updating the selected menu item in the BottomNavigationView.

But What if you want to do any of the following case scenarios:
1. Allow menu items to have other functionalities rather than only moving through the available navigation destinations
2. Applying Custom Transition Animation

# Case Scenario One
For the first case scenario, you may have 5 menu items in the BottomNavigationViews. Four of them is used to navigate between available destinations and the last one is used to start a new activity (maybe a SettingsActivity). What will come up on your mind is to use the setOnNavigationItemSelectedListener() method. Yes this is the first step of the solution, But actually you are breaking the communications between the NavController and BottomNavigationView which has been handled automatically as you called 

        NavigationUI.setupWithNavController(bottomNav, navController);
 
So you have to drop using this simple linking method and use another method which is

        NavigationUI.onNavDestinationSelected(item, navController);

Here is the Steps to achieve the desired behavior in the first case scenario;
1. Use setOnNavigationItemSelectedListener() method to control what happen when you select a menu item
2. Check if the NavGraph contains a similar destination as the menu item id
3. For menu items that used to navigate between the NavGraph destination, Use NavigationUI.onNavDestinationSelected(item, navController) to handle the navigation for you.
4. If there is no such destination, Use Switch/Case block to differentiate between the menu items
5. Always return true so that the BottomNavigationView item is checked. 
Note: if you want to use the settings item to open a SettingsActivity, it is better to return false so that the Settings Item is not get checked So when you press the back button and return back to the MainActivity you will find the right menu item checked according to the last position before leaving the activity.
6. Override the onBackPressed() method to check the menu item related to the start destination fragment.

Here is a complete code snippet

    @Override
    protected void onCreate(Bundle savedInstanceState) {
    ....
        NavController navController = Navigation.findNavController(this, R.id.nav_host_fragment);
            
        bottomNav.setOnNavigationItemSelectedListener(new BottomNavigationView.OnNavigationItemSelectedListener() {
            @Override
            public boolean onNavigationItemSelected(@NonNull MenuItem item) {
                boolean handled = false;
                if (navController.getGraph().findNode(item.getItemId()) != null) {
                    NavigationUI.onNavDestinationSelected(item, navController);
                    handled = true;
                } else {
                    switch (item.getItemId()){
                        case R.id.settings:
                            openSettingsActivity();
                            break;
                    }
                }

                return handled;
            }
        });
    }

    @Override
    public void onBackPressed() {
        bottomNav.getMenu().getItem(0).setChecked(true);
        super.onBackPressed(); 
    }

# Case Scenario Two
For this case scenario, you have the exact first scenario but you also want to apply a custom transition animation. Fortunately, Android Jetpack Navigation Component allows you to create custom transition animation but unfortunately you don't have a direct api to add these custom transition animation to the NavigationUI.setupWithNavController() api. Also, NavigationUI.onNavDestinationSelected() method doesn't support custom transition animation So, you have to use another api which allow you to add these custom transition animations.

Here is the Steps to achieve the desired behavior in the first case scenario;
1. Create a NavOption object and pass the custom transition animation to it using the following methods

                .setEnterAnim(R.anim.slide_in_right)
                .setExitAnim(R.anim.slide_out_left)
                .setPopEnterAnim(R.anim.slide_in_right)
                .setPopExitAnim(R.anim.slide_out_left)
        
2. You have to make sure of creating only one instance of the same destination target (the destination fragment you are navigating to) by adding this parameter to the NavOption Object

                .setLaunchSingleTop(true)
                
3. You will also need to make sure that the navigation BackStack is cleared with every transition so that when you press the back button, you go for the start destination fragment, not the previous selected fragment. This can be achieved by adding the following parameters to the NavOption Object

                .setPopUpTo(navController.getGraph().getStartDestination(), false)

4. Pass the NavOption Object to the NavController.navigate() method to be applied for the navigation operation
                    navController.navigate(item.getItemId(), null, navOptions);
5. As Previous Case Scenario, you have to Use setOnNavigationItemSelectedListener().
6. Check if the NavGraph contains a similar destination as the menu item id
7. For menu items that used to navigate between the NavGraph destination, Use 
navController.navigate(item.getItemId(), null, navOptions) to handle the navigation for you.
8. If there is no such destination, Use Switch/Case block to differentiate between the menu items
9. Always return true so that the BottomNavigationView item is checked. 
10. Override the onBackPressed() method to check the menu item related to the start destination fragment.

Here is a complete code snippet


    @Override
    protected void onCreate(Bundle savedInstanceState) {
    ....
        NavController navController = Navigation.findNavController(this, R.id.nav_host_fragment);

        NavOptions navOptions = new NavOptions.Builder()
                .setLaunchSingleTop(true)
                .setEnterAnim(R.anim.slide_in_right)
                .setExitAnim(R.anim.slide_out_left)
                .setPopEnterAnim(R.anim.slide_in_right)
                .setPopExitAnim(R.anim.slide_out_left)
                .setPopUpTo(navController.getGraph().getStartDestination(), false)
                .build();

        bottomNav.setOnNavigationItemSelectedListener(new BottomNavigationView.OnNavigationItemSelectedListener() {
            @Override
            public boolean onNavigationItemSelected(@NonNull MenuItem item) {
                boolean handled = false;
                if (navController.getGraph().findNode(item.getItemId()) != null) {
                    navController.navigate(item.getItemId(), null, navOptions);
                    handled = true;
                } else {
                    switch (item.getItemId()) {
                        case R.id.settings:
                            openSettingsActivity();
                            break;
                    }
                }

                return handled;
            }
        });
    }


    @Override
    public void onBackPressed() {
        bottomNav.getMenu().getItem(0).setChecked(true);
        super.onBackPressed(); 
    }

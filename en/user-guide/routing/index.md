>ViewModel first navigation is currently in discussion how to improve the current implmentation.
The following is a transcript of a Slack discussion on this topic:

jcmm33 [10:33 AM] 
@aalmada i can tell you what we do for vm based routing / navigation if you are interested, if that works for xam forms I’ve no idea, but it does work for android/iOS & Win* solutions (edited)

 

aalmada [10:51 AM] 
@jcmm33 RxUI documentation lists xam forms with an "absolutely" for vm-based routing https://docs.reactiveui.net/en/user-guide/routing/

[10:52]  
It should be easy but I'm having trouble with VM constructors and navigation in commands.

jcmm33 [10:57 AM] 
@aalmada we abandoned trying to use the existing stuff for navigation/presentation a long time ago and instead did our own thing.


aalmada [11:02 AM] 
@jcmm33 That same documentation does list your platforms as problematic. I'd like to contribute documenting this feature so I want to use what's available in RxUI. Thanks for the offer to help anyway.


escamoteur [2:01 PM] 
@aalmada you are completely right RyUI navigation for XF is limited. Unless we can overhaul the whole naviation part of RxUI I recommend having a look at xamvvm together with RxUI https://github.com/xamvvm/xamvvm/wiki/ReactiveUI-in-unison-with-xamvvm
GitHub
xamvvm/xamvvm
xamvvm - Simple MVVM (Model, ViewModel, View) Framework for .Net - Xamarin.Forms compatible 
 
 
ghuntley [2:30 PM] 
@aalmada @escamoteur viewmodel navigation complete overhaul is in the roadmap for this year - jcm33 will most likely be design lead and help others implement as he's adhead of us and has a heap of knowledge/lessons learned we can use to our advantage. (edited)


ghuntley [3:24 PM] 
replied to a thread
ghuntley 2:21 PM
Yeah we feel that vm2vm is lacking in the framework/on the TODO  (and I'm hoping @jcmm33 will eventually be able to contribute some of his private work in this arena sometime in the future)
https://reactivex.slack.com/archives/reactiveui/p1485868893003544
5 replies
ghuntley 3:24 PM
Can't wait. Didn't know about https://github.com/VistianOpenSource NICE.
https://reactivex.slack.com/archives/reactiveui/p1486391095003967?thread_ts=1485868893003544&cid=C02AJB872

marcbruins [3:29 PM] 
@jcmm33 i would like to help out with vm to vm navigation as wel! Done a lot of presenters in MvvmCross, its the approach you head in mind the same way?

jcmm33 [3:33 PM] 
@marcbruins yes, we don’t refer to it as navigation but instead presentation. We have presenters which you inject in, for which there is a ‘default set' of moving to controllers/activities by having a request and the request is unpacked for the associated view model. We try however to not inject this as such but instead inject simple presenters to the view models which invoke methods like ‘PresentSearchResultsFor..’ , these then typically use the ‘default set’ to achieve what is required.

[3:34]  
@marcbruins we’ve also got mechanisms for getting return values back easily through observables, and will probably include ReactiveUI interactions into the same mechanism so we have just one way of ‘presenting'

marcbruins [3:37 PM] 
Thats really interesting @jcmm33 is there a sample somewhere?

jcmm33 [3:38 PM] 
@marcbruins I can post some samples . Slowly going through process of removing internal dependancies so can make it public. A lot of the other packages, we’ve made available, just need to get a get the rest of the work done now.


jcmm33 [3:43 PM] 
Thing is we’ve built out the activity/controller classes a fair bit to make things simpler for us e.g. a lot of common things can be specified by attributes on activity class , layout, drawerlayout, menus etc. We’ve also built out a couple of ‘base’ view models which also allow for composition through the specification of attributes. So for example one of our attributes is a command logging behavior which automatically logs whenever a command is executed.

escamoteur [3:44 PM] 
how good will this translate to iOS or XF?

jcmm33 [3:54 PM] 
it works fine with iOS & Android. We have a ‘deprecated’ WinRT code base - we gave in on this since Microsoft gave in with it!

[3:56]  
My knowledge of Xam forms isn’t great, rather non existent, but if navigation to views is done through either creating instances directly or requesting by ‘type’ and its possible to plumb into a very early event in the view class then it shouldn’t be that difficult to do.

escamoteur [3:58 PM] 
ok. Guess I will have so see some code to get am impression

giusepe [4:07 PM] 
I’m on same boat as @escamoteur: Seems nice, but I would be lying if I tell you that I completely understood.

from @jcm33

## Sample View Model Navigation
Last edited 4 days ago
We don't typically think of it as View Model navigation but instead View Model presentation, since the behaviour could well change between platforms and doesn't neccessarily mean that one is infact navigating to a completely new view.
A typical presenter used by a View Model would look like this.
This is our presenter for the introduction phase of our application.
/// <summary>
/// Initial Introduction Presenter.
/// </summary>
[Preserve(AllMembers = true)]
public class StickxIntroductionPresenter : IIntroductionPresenter
{
	private readonly IPresentationFrame _frame;

	public StickxIntroductionPresenter(IPresentationFrame frame)
	{
		_frame = frame;
	}
	public IObservable<bool> PresentFacebookSignIn()
	{
		return _frame.Open(new RxTypePresentRequest<FacebookViewModel>());
	}

	public IObservable<bool> PresentUserInterestsSelections()
	{
		return _frame.Open(new RxTypePresentRequest<UserInterestsViewModel>(presentationHint: NavigationViewPresentationHint.NavigationReplace));
	}

	public IObservable<bool> PresentSpecifyInitialLocation()
	{
		return _frame.Open(new RxTypePresentRequest<SpecifyLocationViewModel>(presentationHint: NavigationViewPresentationHint.NavigationReplace));
	}
}
This (well the interface IIntroductionPresenter) would typically be injected automatically into the constructor of the view model(s) that wish to present. In this case, the implementation utilizes the 'default' presentation frames, for which an implementation already exists per each platform.
Presentation requests can be comprised of :
Simplistic by 'type', with automatic use of IoC to construct the view model.
By 'type' with initialization parameters (both method invocation & property assignment available).
Existing view model instances
Custom 'request' types can be easily constructed and deployed, should it be desired.
Important : It should be noted that our approach works by us having our own base classes from Activities,Fragments & Controllers which we derive from, and we operate on the assumption that injected parameters in View Model constructors are NOT specific to the present request being made;the parameters for the present request under these scenarios are provided either through an initialization method and/or through properties of the View Model.
The default presentation frame for each platform allows for hints to be provided customize how things are presented e.g. remove all existing views, push onto navigation stack,replace top item only with this etc.
The default presentation frame for each platform allows for present requests to also await a response from the presented view - somewhat akin to interactions.
Platform specific attributes can also be passed through as an "agnostic" bag which the receiver understands and knows how to handle (think of the content transitions in Android).
A low presentation frame has to implement (for which Android,iOS & WinRT versions already exist):
/// <summary>
/// Specification for the presenter of views in a frame given a <see cref="RxPresentRequest"/>
/// </summary>
public interface IPresentationFrame
{
	/// <summary>
	/// Present the view for the specified request.
	/// </summary>
	/// <param name="presentRequest"></param>
	/// <returns></returns>
	IObservable<bool> Open(RxPresentRequest presentRequest);

	/// <summary>
	/// Present the view for the specified request and return the obtained value.
	/// </summary>
	/// <typeparam name="T"></typeparam>
	/// <param name="presentRequest"></param>
	/// <returns></returns>
	IObservable<Result<T>> OpenWithResult<T>(RxPresentRequest presentRequest);
It should be noted that one doesn't have to create presenters, one could just use the lower level frame present request constructs, but we've found that adding an interface for the present operations which a view model uses makes it very easy to unit test and better reflects the intent the view model has.
Presentation doesn't have to revolve just around a navigation between activitives / controllers. They can also be used for popup constructs.


@escamoteur @marcbruins @giusepe A small background on what we do regarding navigation/presentation.


giusepe [7:14 PM] 
My brain hurts... I'll read it again :slightly_smiling_face:

pureween [8:13 PM] 
So for XF it seems like it's less about iOS vs Android vs WinRT and more about what XF navigation idea you're connecting the Presenter over as XF has already abstracted the native navigation.... Like  on Android, XF just uses a single activity opposed to navigating between activities.  So basically you'd specify the XF navigation paradigm you are using  (MasterDetails vs just a NavigationPage vs Tabbed Pages vs Other ),  then when you indicate a "Presentation" you want to get to that presentation gets connected over your XF Navigation paradigm

so like in our app we have it setup so you can basically inject into the presenter whether it's using a MasterDetails or just a NavigationPage and then when those presentations are requested they get filled in accordingly but we aren't really paying attention to iOS vs Android vs WinRT

The only place where we have to pay attention is when XF doesn't have an XF default set of something like say popping open an input box... Those are cases when we then defer to the platform to implement it specifically. But typically we use XF to try and fulfill the " ‘default set’ to achieve what is required."  but then that default set can be elaborated on if needed.... I feel like I'm basically just describing how navigation in Prism works :slightly_smiling_face:

ghuntley [8:55 PM





# Routing / Navigation

https://stackoverflow.com/questions/26898381/reactiveui-view-viewmodel-injection-and-di-in-general

View-First or ViewModel-First?

Whether you can use VM-based routing (i.e. RoutedViewHost, IScreen, RoutingState, and friends) in ReactiveUI depends on the platform you're on:

* WPF, Xamarin Forms: Absolutely
* WP8, WinRT: You can make it work, you lose some transitions and niceties
* Android, iOS Native: Very difficult to make work


# Wizard Navigation

    phil.cleveland [8:16 AM] 
    @moswald: I see how that could work.  I give up the default binding goodness of the command

    phil.cleveland [8:18 AM]
    @michaelteper: Not sure I follow where you are indicating to put those subjects

    phil.cleveland [8:19 AM]
    My original design had each page impl the Next and Back and also the xaml for the buttons.  I didn't like it because each page then had to know about all the other pages it could possibly go to or go back to.  So I moved all the logic to the shell, but that forces me to recreate the command each time a page changes. (Which I think is legit, but doesn't work)

    moswald [8:20 AM] 
    well, my solution is a little bit wrong, combine it with @michaelteper's

    moswald [8:20 AM]
    pass two `Subject<bool>`s into your page VMs

    moswald [8:20 AM]
    those subjects are your `CanExecute`s

    moswald [8:21 AM]
    that way you keep your `ReactiveCommand` binding goodness

    phil.cleveland [8:21 AM] 
    I see. So set up the cmd with those and then the pages OnNext to define the enabled

    phil.cleveland [8:21 AM]
    Ok.  I like that

    phil.cleveland [8:22 AM]
    Thanks :simple_smile:

    michaelteper [8:23 AM] 
    ```var canGoBack = new Subject<bool>();
    var canGoNext = new Subject<bool>();
    BackButton = ReactiveCommand.Create(canGoBack);
    NextButton = ReactiveCommand.Create(canGoNext);
    this.WhenAnyValue(x => x.Router.CurrentViewModel)
                    .Subscribe(cvm =>
                    {
                        var page = Router.GetCurrentViewModel() as IWizardPage;
                        canGoBack.OnNext(page.CanMoveBack);
                        canGoNext.OnNext(page.CanMoveNext);

              ...

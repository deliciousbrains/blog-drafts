# Post GUIDs: Sometimes You Should Update Them

***TL;DR** &mdash; Updating the GUIDs of a WordPress site that's already live will annoy people subscribed to its RSS feeds. But if the site has never been live, you should update the GUIDs.*

#### What is a GUID?

GUID is an acronym for Globally Unique Identifier. It is a concept that goes well beyond WordPress and is used in all kinds of applications. In the past, I used GUIDs heavily working with ASP.NET and Microsoft SQL Server for example. Those GUIDs looked like this:

```
6F9619FF-8B86-D011-B42D-00C04FC964FF
```

(I know, shocking right! A Microsoft stack, really dude? If it's any consolation, I hated every minute of it.)

RSS feeds are another example that employ GUIDs. Every item in an RSS feed can have a `<guid>` element.

#### How does WordPress use GUIDs?

WordPress employs GUIDs when it generates its RSS (and Atom) feeds. If you take a look at the [RSS feed of this blog](https://deliciousbrains.com/blog/feed/) you will see elements similar to this:

```
<guid isPermaLink="false">https://deliciousbrains.com/?p=6806</guid>
```

WordPress uses a real, working post URL as the GUID. In fact, if you go to the URL in the example above, it will redirect you to the blog post it belong to. This is also the URL the post has when using the default Permalink setting. Since URLs are globally unique by their nature, they make pretty good GUIDs so long as they don't change. And in this case, since we're using the post ID in the URL, it's almost certain not to change. A good design decision.

#### What are GUIDs used for in RSS feeds?

RSS feed readers (like Feedly) use GUIDs to uniquely identify an item against all other items that have appeared in the feed and will appear in the feed in the future. It uses the GUID to determine if it should show an item as new.

The GUID is supposed to be the constant. Other details of the post may change, like the title, content, URL, dates, etc but so long as the GUID stays the same, the feed reader can avoid showing the same item twice.

For example, let's say we update the URL of one of our posts from `https://deliciousbrains.com/tour-of-the-wordpress-database/` to `https://deliciousbrains.com/tour-wordpress-database/`. If the RSS feed was just looking at the URL, it would see two posts and show two items with the same title and content in the reader. But since it looks at the GUID and the GUID hasn't changed, it knows that it's the same post.

#### Why would I change the GUIDs?

Accidentally. You might not realize you have.

People often tell me they don't need WP Migrate DB. When they need to migrate a site from one domain to another, they just dump their database to an SQL file, open it in a text editor, and run a find & replace.

There are several reasons I don't recommend this, but let's focus on GUIDs. We've already established that WordPress uses URLs for GUIDs. Therefore, doing a domain find & replace on an SQL file will almost certainly change the GUIDs.

And if you change the GUIDs, all of the posts in your RSS feed will show up as new posts in your subscribers' feed reader, annoying them at best and certainly giving them a reason to unsubscribe.

#### Should you ever change GUIDs?

Yes, there are some cases when you probably should update the GUIDs. For example, if you're migrating from a dev or staging site to live and the site has never been live before. You don't really want your GUIDs to be a dev or staging URL. It probably wouldn't hurt anything, but it certainly isn't reassuring to see dev and staging URLs in a live database.

That's why we give you the option in WP Migrate DB (and Pro) to replace GUIDs or not:

![Replace GUIDs setting in WP Migrate DB](https://s3.amazonaws.com/cdn.deliciousbrains.com/content/uploads/2015/03/Screen-Shot-2015-03-01-at-4.30.01-PM.png)

How do you make sure that your GUIDs are updated only when they should be? Do you use WP Migrate DB? Did you know about this feature?

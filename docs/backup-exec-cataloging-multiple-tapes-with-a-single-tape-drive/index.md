# Backup Exec – cataloging multiple tapes with a single tape drive

Recently I&#8217;ve had a request to restore a certain backup from the old backup tapes. We&#8217;ve had a new Backup exec installation which didn&#8217;t have the catalogue/record of these tapes nor the actual tape drive to recover them &#8211; I knew it was going to be one in the world task. And the hell it was.

Locating tapes in the storage unit was a pure joy &#8211; no need to go to the gym after this. Buying some used tape drive and set this up to the was <g class="gr_ gr\_82 gr-alert sel gr\_spell gr\_replaced gr\_inline\_cards gr\_disable\_anim\_appear ContextualSpelling ins-del multiReplace" id="82" data-gr-id="82">surprisingly</g> easy.

So I had a working Backup exec &#8211; check &#8211; I had the backup tapes &#8211; check &#8211; and working backup drive &#8211; check check check. I&#8217;d expected only lollipops and unicorns from that point on, and I couldn&#8217;t be more wrong.

Cutting the long and <g class="gr_ gr\_5 gr-alert sel gr\_spell gr\_replaced gr\_inline\_cards gr\_disable\_anim\_appear ContextualSpelling ins-del multiReplace" id="5" data-gr-id="5">frustrating</g> story short &#8211; here are the steps needed to catalogue all tapes in one go. 

## Disable settings

You&#8217;d like to go to Backup Exec settings> Cataloging tab

Then untick both boxes: &#8220;Request all media in the sequence for <g class="gr_ gr\_15 gr-alert gr\_spell gr\_inline\_cards gr\_run\_anim ContextualSpelling multiReplace" id="15" data-gr-id="15">catalog</g> operations&#8221; and &#8220;Use storage-based <g class="gr_ gr\_31 gr-alert gr\_spell gr\_inline\_cards gr\_run\_anim ContextualSpelling multiReplace" id="31" data-gr-id="31">catalogs</g>&#8220;

<div class="wp-block-image">
  <figure class="aligncenter"><img loading="lazy" width="295" height="29" src="https://i2.wp.com/kamilpro.com/wp-content/uploads/2018/09/image.jpg?resize=295%2C29&#038;ssl=1" alt="" class="wp-image-1300" data-recalc-dims="1" /><figcaption>Backup exec catalogue settings</figcaption></figure>
</div>

Making this two changes will allow you to catalogue &#8220;some&#8221; of the backup tapes but I&#8217;ve found a weird glitch here &#8211; say you have a set of 4 backup tapes. Then obviously you will &#8220;Inventory & Catalog&#8221; the first tape, <g class="gr_ gr\_202 gr-alert sel gr\_spell gr\_replaced gr\_inline\_cards gr\_disable\_anim\_appear ContextualSpelling ins-del" id="202" data-gr-id="202">the</g> second and so on&#8230;

The problem with this take is that each subsequent tape will remove <g class="gr_ gr\_6 gr-alert sel gr\_spell gr\_replaced gr\_inline\_cards gr\_disable\_anim\_appear ContextualSpelling multiReplace" id="6" data-gr-id="6">catalogue</g> information of the previous tape &#8211; thus you end up with information only about the very last backup tape.

To fix that is as simple as

## Inventory all backup tapes prior cataloguing them

Yep, that&#8217;s it. For some reason without that Backup exec seems to be unaware of any following tapes, and thus is not requesting them. When you actually inventory all of them before then Backup exec will keep requesting them during single <g class="gr_ gr\_298 gr-alert sel gr\_spell gr\_replaced gr\_inline\_cards gr\_disable\_anim\_appear ContextualSpelling multiReplace" id="298" data-gr-id="298">catalogue</g> job.

Bear in mind, the server will update its records only after completing the catalogue job, therefore _patience you must have, my young sysadmin._


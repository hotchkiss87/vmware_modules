# Greetings! #
These changes fix vmware's modules on new kernels.

It was published on the technical blog <http://rglinuxtech.com/?p=1706>.

The patch I left in plain text on the blog was formatted in a weird way, but I was also able to
describe what the patch was doing in a way that it could be applied by anyone.

The issue was related to commit c12d2da56d0e07d230968ee2305aaa86b93a6832 where there were changes
to the get_user\*\(\) APIs.

Link to kernel sources: <https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/>

The related commit text is:

```
commit c12d2da56d0e07d230968ee2305aaa86b93a6832
Author: Ingo Molnar <mingo@kernel.org>
Date:   Mon Apr 4 10:24:58 2016 +0200

    mm/gup: Remove the macro overload API migration helpers from the get_user*() APIs

    The pkeys changes brought about a truly hideous set of macros in:

      cde70140fed8 ("mm/gup: Overload get_user_pages() functions")

    ... which macros are (ab-)using the fact that __VA_ARGS__ can be used
    to shift parameter positions in macro arguments without breaking the
    build and so can be used to call separate C functions depending on
    the number of arguments of the macro.

    This allowed easy migration of these 3 GUP APIs, as both these variants
    worked at the C level:

      old:
            ret = get_user_pages(current, current->mm, address, 1, 1, 0, &page, NULL);

      new:
            ret = get_user_pages(address, 1, 1, 0, &page, NULL);

    ... while we also generated a (functionally harmless but noticeable) build
    time warning if the old API was used. As there are over 300 uses of these
    APIs, this trick eased the migration of the API and avoided excessive
    migration pain in linux-next.

    Now, with its work done, get rid of all of that complication and ugliness:

        3 files changed, 16 insertions(+), 140 deletions(-)

    ... where the linecount of the migration hack was further inflated by the
    fact that there are NOMMU variants of these GUP APIs as well.

    Much of the conversion was done in linux-next over the past couple of months,
    and Linus recently removed all remaining old API uses from the upstream tree
    in the following upstrea commit:

      cb107161df3c ("Convert straggling drivers to new six-argument get_user_pages()")

    There was one more old-API usage in mm/gup.c, in the CONFIG_HAVE_GENERIC_RCU_GUP
    code path that ARM, ARM64 and PowerPC uses.

    After this commit any old API usage will break the build.

    [ Also fixed a PowerPC/HAVE_GENERIC_RCU_GUP warning reported by Stephen Rothwell. ]

    Cc: Andrew Morton <akpm@linux-foundation.org>
    Cc: Dave Hansen <dave.hansen@linux.intel.com>
    Cc: Dave Hansen <dave@sr71.net>
    Cc: Linus Torvalds <torvalds@linux-foundation.org>
    Cc: Peter Zijlstra <peterz@infradead.org>
    Cc: Stephen Rothwell <sfr@canb.auug.org.au>
    Cc: Thomas Gleixner <tglx@linutronix.de>
    Cc: linux-kernel@vger.kernel.org
    Cc: linux-mm@kvack.org
    Signed-off-by: Ingo Molnar <mingo@kernel.org>
```

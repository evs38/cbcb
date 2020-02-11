# cbcb
Content-Blind Cancelbot for Usenet

      			  by Dr.Dimitri Vulis (KOTM)

               A Content-Blind Cancelbot for Usenet (CBCB)

Usenet News is a popular system for transmitting articles. Historically it
used to propagate over UUCP. However today most of the transmission is done
over the Internet TCP/IP connections using the NNTP protocol (RFC 977).

Each article consists of a series of headers of the form
Keyword: value
followed by a blank line, followed by the body of the message.
Some required headers are self-explanatory: From:, Date:, Subject:.

The Newsgroups: header identifies a series of keywords that can be used
to search for articles in the newsfeed. For example:
Newsgroups: news.admin.policy,comp.lang.c
identifies a Usenet article relevant to both Usenet administrative policy
and to the C computer language.

The Message-Id: header uniquely identifies each article. For example:
Message-Id: <12341223@whitehouse.gov>
The message-ids are not supposed to be recycled.

The cancelbot program is supposed to search the user-specified newsgroups for
articles whose headers match user-specified regular expressions and to issue
special 'cancel' control articles. It will copy some of the headers from the
original message and add a special header:
Control: cancel <message-id>

This program is an NNTP client. Much of the processing is offloaded to an
NNTP server, to which the cancelbot talks using the Internet sockets protocol.

This cancelbot does not look at article bodies and is therefore content-blind.

Inputs:

`argv[1] (required) hosts file`

A line that starts with # is a comment. Otherwise, each line contains the
following 5 fields:

1. hostname (some.domain.com) or ip address (a.b.c.d)
2. port (normally 119)
3. Y/N - do we ask this host for NEWNEWS/HEADER?
4. I/P/N - do we inject cancels to this host with IHAVE, POST, not at all
5. Timeout - the number of seconds to wait for a response from this server.

Example of a hosts file:

```
# ask the local server for new news and post back the cancels
127.0.0.1 119 Y P 60
# don't get message-ids from remote server, but give it cancels via IHAVE
news.xx.net 119 N I 300
```
argv[2] (required) target file

A line that starts with # is a comment. Otherwise, each line contains the
following 9 fields:

1. List of newsgroups to be scanned for new messages. This is not interpreted
by the cancelbot, but passed on to the NNTP server. Per RFC 997, multiple
groups can be separated by commas. Asterisk "*" may be used to match multiple
newsgroup names. The exclamation point "!" (as the first character) may be used
to negate a match. Warning: specifying a single * will generate a lot of data.

Example: news.groups,comp.*,sci.*,!sci.math.*

2. A watchword (case-sensitive) that needs to be contained in the article
headers for the cancel to be issued.

3. Format of the Subject: header in the cancel article.
 C - Subject cancel <message-id> (same as Control:)
 O - Subject: header copied from the original article
 N - none.
If N is specified, then Subject: MUST be provided in the file appended to
the header, or the cancel won't propagate.

4. cancel message-id prefix
 normally cancel. or cn.

Most cancellation articles follow the so-called $alz convention:
Control: cancel <message.id>
Message-id: <cancel.message.id>
However this is not a requirement.

5. path constant (string to put in path). May be 'none'.
6. path copy # (number of elements to copy from the right, may be 0)

Explanation of these two parameters:
each Usenet article contains the "Path:" header with a list of hosts separated
by explanation marks. For example:
Path: ohost1!ohost2!ohost3!ohost4
If you specify path constant of "nhosta!nhostb" and path copy of 2
then the path written by cbcb will be
Path: nhosta!nhostb!ohost3!ohost4

7. Name of the file appended to the header or 'none'

Examples:
```
# should be supplied as a courtesy
X-Cancelled-By: Cancelbot
# if and only if target file field 3 contains 'N':
Subject: Cancelling a Usenet article
# only if posting via IHAVE:
NNTP-Posting-Host: usenet.cabal.org
```
8. Name of the file that will become the body of the cancel or 'none'

If 'none' is specified, the default will be
"Please cancel this article."

9. The string to be prepended to the newsgroups. Normally 'none',
but may be set to something like misc.test (or misc.test,alt.test).

Example of a target file:
```
# delete all articles that mention C++ (but not c++)
comp.lang.c.* C++ C cancel. cyberspam 3 can.hdr none none
# no sex in the sci hierarchy, and add misc.test to the cancel
sci.* sex C cn. plutonium 2 can1.hdr can.txt misc.test
```
argv[3] (optional) datestamp, YYMMDD. If not specified, default is 900101. Only
articles after this date are examined. This parameter is not processed by the
cancelbot, but passed on to the NNTP server. It should normally be specified
so as not to look at old Usenet articles.

argv[4] (optional) timestamp, digits HHMMSS, where HH is hours on the 24-hour
clock, MM is minutes 00-59, and SS is seconds 00-59. If not specified, default
is 000000. Note that both datestamp and timestamp are in Greenwich mean time.

To compile, you must define an OS type (under gcc, this is accomplished using
the -Dmacro directive).  Under Unix, for example:
gcc -DCBCB_UNIX -o cancelbot cbcb.c

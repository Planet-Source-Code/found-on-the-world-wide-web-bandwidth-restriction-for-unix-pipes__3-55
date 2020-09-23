<div align="center">

## Bandwidth restriction for UNIX pipes


</div>

### Description

Slowpipe allows the restriction of bandwidth on a modem network connection where a Unix pipe may be used. It was written to address a problem for users of a small network connected to a larger network with a low bandwidth connection.

If you use such a connection for both interactive work, and transfer of large files it is possible that using programs such as ftp will consume all available bandwith, making the use of terminal emulators, X-windows etc., near impossible until the transfer is complete.

Slowpipe is a simple pipe filter intended to pass all characters through unchanged, but to limit the transfer rate. It does not however have any special knowledge of TCP/IP or other protocols. You may wish to change the packet sizes to multiples of the MTU to your host network for optimal performance if you are able to establish this. The default values though have been found to work satisfactorily on more than one network. The ideal solution is to reconfigure the router to your host network to restrict bandwidth on specific ranges of TCP/IP port numbers though this was not possible in my case.

Slowpipe has been tested under Linux and AIX and appears to perform as expected - though I offer no guarantees as this is free software. The user of the software must test their build. No liabilty shall be accepted for failure to perform as expected or consequential damages.

I emphasise that this is a simple solution and better solutions may exist - but it solved our problem.

http://www.birdsoft.demon.co.uk/proglib/slowpipe.htm
 
### More Info
 
I emphasise that this is a simple solution and better solutions may exist - but it solved our problem.

The timing loop assumes that it takes zero time to read from the standard input. You will find the actual transfer rates to be marginally incorrect, though they are accurate enough to provide control.

Transfer rates are specified in kilobytes per second. As a guidline, try about one third of the rate you can achieve with ftp.

Example: to copy a directory tree to the home directory on a remote system

tar cvf - | slowpipe 2.0 | rsh remotehost tar xvf -

where remotehost is the name of the remote system


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[Found on the World Wide Web](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/found-on-the-world-wide-web.md)
**Level**          |Unknown
**User Rating**    |5.0 (10 globes from 2 users)
**Compatibility**  |C, C\+\+ \(general\)
**Category**       |[System Services/ Functions](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/system-services-functions__3-23.md)
**World**          |[C / C\+\+](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/c-c.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/found-on-the-world-wide-web-bandwidth-restriction-for-unix-pipes__3-55/archive/master.zip)





### Source Code

```
/*
 * Title: 	Slowpipe.C
 *
 * Description: Bandwidth restriction for UNIX pipes
 *
 * This software may be freely redistrubuted providing this comment remains
 * unchanged.
 *
 * Author: 	Iain W. Bird, http://www.birdsoft.demon.co.uk
 *		wes@birdsoft.demon.co.uk
 */
#include <sys/time.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
main(int argc, char **argv)
{
	FILE *fd;
	char c;
	char *buffer;
	int bufsiz = 16;
	int i;
	int sleep = 0;
	int full = 0;
	struct timeval tval;
	double kps = 2.0,bps,pps;
	unsigned long t_s, t_us;
	if(argc != 2)
	{
		perror("Usage slowpipe <K per second e.g. 6.0>");
		exit(1);
	}
	sscanf(argv[1],"%lg",&kps);
	if ( kps > 10.0 )
	{
		bufsiz = 1024;
	}
	else if ( kps > 5.0 )
	{
		bufsiz = 512;
	}
	else if ( kps > 2.0 )
	{
		bufsiz = 256;
	}
	else if ( kps > 1.0 )
	{
		bufsiz = 128;
	}
	else if ( kps > 0.5 )
	{
		bufsiz = 64;
	}
	bps = 1024.0 * kps;
	pps = bps / bufsiz;
	if(pps > 1.0)
	{
		t_s = 0.0;
		t_us= 1.0e6 / pps;
	}
	else
	{
		t_s = 1.0 / pps;
		t_us = 0;
	}
	fprintf(stderr,"%6.2g K per second, bufsiz = %d\n",kps,bufsiz);
	buffer = malloc(bufsiz);
	if(!buffer)
	{
		perror("Unable to allocate buffer");
		exit(1);
	}
	i = 0;
	while (1)
	{
		if (!full)
		{
			if(i < bufsiz)
			{
				buffer[i] = fgetc(stdin);
				if(feof(stdin))
				{
					fwrite(buffer,1,i,stdout);
					break;
				}
				if(++i == bufsiz)
				{
					full = !0;
				}
			}
		}
		else
		{
			/* go for a blocking select since the buffer is full */
			tval.tv_sec = t_s;
			tval.tv_usec = t_us;
			select ( 0, 0, 0, 0, &tval );
			fwrite(buffer,1,bufsiz,stdout);
			full = 0;
			i = 0;
		}
	}
	free(buffer);
}
```


title: springmvc自定义实现下载视图view 

#  SpringMVC自定义实现下载视图View 
注意：View与Controller不同，View不是单例的，每次通过ViewResolver解析出View或者手动创建都会得到一个新的对象。
```

import org.springframework.web.servlet.View;

public class DownloadingView implements View {
	private final File dfile;
	private final String contentType;

	public DownloadingView(File dfile, String contentType) {
		this.dfile = dfile;
		this.contentType = contentType;
	}

	@Override
	public String getContentType() {
		return this.contentType;
	}

	@Override
	public void render(Map<String, ?> model, HttpServletRequest request, HttpServletResponse response)
			throws Exception {
		String fileName = dfile.getName();
		// 避免下载时文件名乱码
		fileName = new String(fileName.getBytes(), "ISO-8859-1");
		response.setHeader("Content-Disposition", "attachment; filename=" + fileName);
		response.setContentType("application/octet-stream");
		response.setContentLength((int) dfile.length());
		// 得到输入流
		FileInputStream in = new FileInputStream(dfile);
		BufferedInputStream bufIn = new BufferedInputStream(in);
		ServletOutputStream out = response.getOutputStream();
		// 下面是一个普通的流的复制 。。。忽略 .这样可以防止内存问题
		byte[] bs = new byte[1024];
		int len = 0;
		while ((len = bufIn.read(bs)) != -1) {
			out.write(bs);
		}
		// 最后是流的关闭。
		out.flush();
		bufIn.close();
		in.close();
	}
}

```
```

 @RequestMapping(
            value = "/{ticketId}/attachment/{attachment:.+}",
            method = RequestMethod.GET
    )
    public View download(@PathVariable("ticketId") long ticketId,
                         @PathVariable("attachment") String name)
    {
        Ticket ticket = this.ticketDatabase.get(ticketId);
        if(ticket == null)
            return this.getListRedirectView();

        Attachment attachment = ticket.getAttachment(name);
        if(attachment == null)
        {
            log.info("Requested attachment {} not found on ticket {}.", name, ticket);
            return this.getListRedirectView();
        }

        return new DownloadingView(new File(attachment.getPath()),
                attachment.getMimeContentType());
    }

```
参考：http://my.oschina.net/HeliosFly/blog/221392
http://www.blogjava.net/wangajing/archive/2009/11/26/303759.html
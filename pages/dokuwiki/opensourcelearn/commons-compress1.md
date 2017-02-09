title: commons-compress1 

#  commons-compress压缩文件入门 
```

import java.io.BufferedInputStream;
import java.io.BufferedOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;

import org.apache.commons.compress.archivers.zip.ZipArchiveEntry;
import org.apache.commons.compress.archivers.zip.ZipArchiveInputStream;
import org.apache.commons.compress.archivers.zip.ZipArchiveOutputStream;
import org.apache.commons.io.IOUtils;

/**
 * @功能描述 xxxx<br>
 * 
 * @author dell
 * @创建时间 2016年1月19日 上午11:18:46
 * @History 修订历史<br>
 * 
 */
public class ZipFileUtil {
	/**
	 * 把一个目录打包到一个指定的zip文件中
	 * 
	 * @param dirpath
	 *            目录路径
	 * @param zipPath
	 *            zip文件路径
	 */
	public static void compressFoldToZip(String dirpath, String zipPath) {
		compressFoldToZip(dirpath,zipPath,"");
	}
	/**
	 *  把一个目录打包到一个指定的zip文件中
	 * @param dirpath
	 * @param zipPath
	 * @param entryPath 压缩内文件逻辑路径。如static/
	 */
	public static void compressFoldToZip(String dirpath, String zipPath,String entryPath){
		if(!entryPath.endsWith(File.separator)&&StringUtil.checkNotEmpty(entryPath)){
			entryPath+=File.separator;
		}
		ZipArchiveOutputStream out = null;
		try {
			out = new ZipArchiveOutputStream(new BufferedOutputStream(new FileOutputStream(new File(zipPath))));
			out.setEncoding("UTF-8");
			compressFoldToZip(out, dirpath, entryPath);
		} catch (FileNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} finally {
			IOUtils.closeQuietly(out);
		}
	}
	/**
	 * 把一个目录打包到一个指定的zip文件中
	 * 
	 * @param out
	 * @param dirpath
	 *            目录路径
	 * @param entryPath
	 *            zip中文件的逻辑路径
	 */
	private static void compressFoldToZip(ZipArchiveOutputStream out, String dirpath, String entryPath) {
		InputStream ins = null;
		File dir = new File(dirpath);
		File[] files = dir.listFiles();
		if (files == null || files.length < 1) {
			return;
		}
		try {

			for (int i = 0; i < files.length; i++) {
				// 判断此文件是否是一个文件夹
				if (files[i].isDirectory()) {
					if(files[i].listFiles().length>0){
						compressFoldToZip(out, files[i].getAbsolutePath(), entryPath + files[i].getName() + File.separator);
					}else{
						addFileToZip(files[i], out, entryPath);
					}
				} else {
					addFileToZip(files[i], out, entryPath);
				}
			}
			out.flush();
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		} finally {
			IOUtils.closeQuietly(ins);
		}
	}

	private static void addFileToZip(File file, ZipArchiveOutputStream out, String entryPath) {
		InputStream ins = null;
		try {
			
			String path=entryPath + file.getName();
			if(file.isDirectory()){
				path=formatDirPath(path); //为了在压缩文件中包含空文件夹
			}
			ZipArchiveEntry entry = new ZipArchiveEntry(path);
			entry.setTime(file.lastModified());
			// entry.setSize(files[i].length());
			out.putArchiveEntry(entry);
			if(!file.isDirectory()){
				ins = new BufferedInputStream(new FileInputStream(file.getAbsolutePath()));
				IOUtils.copy(ins, out);
			}
			out.closeArchiveEntry();
		} catch (IOException e) {
			e.printStackTrace();
		} finally {
			IOUtils.closeQuietly(ins);
		}
	}

	/**
	 * 解压zip文件到指定目录
	 * 
	 * @param zipPath
	 * @param destDir
	 */
	public static void unZipToFold(String zipPath, String destDir) {
		ZipArchiveInputStream ins = null;
		OutputStream os = null;
		File zip = new File(zipPath);
		if (!zip.exists()) {
			return;
		}
		File dest = new File(destDir);
		if (!dest.exists()) {
			dest.mkdirs();
		}
		destDir=formatDirPath(destDir);
		try {
			ins = new ZipArchiveInputStream(new BufferedInputStream(new FileInputStream(zipPath)), "UTF-8");
			ZipArchiveEntry entry = null;
			while ((entry = ins.getNextZipEntry()) != null) {
				if (entry.isDirectory()) {
					File directory = new File(destDir, entry.getName());
					directory.mkdirs();
					directory.setLastModified(entry.getTime());
				} else {
					String absPath=formatPath(destDir+entry.getName());
					mkdirsForFile(absPath);
					File tmpFile=new File(absPath);
					os=new BufferedOutputStream(new FileOutputStream(tmpFile));
					IOUtils.copy(ins, os);
					IOUtils.closeQuietly(os);
					tmpFile.setLastModified(entry.getTime());
				}
			}

		} catch (FileNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} finally {
			IOUtils.closeQuietly(ins);
		}
	}
	private static String formatPath(String path){
		path=path.replace('\\', File.separatorChar);
		path=path.replace('/', File.separatorChar);
		return path;
	}
	private static String formatDirPath(String dir){
		if(!dir.endsWith(File.separator)){
			dir+=File.separator;
		}
		return dir;
	}
	private static void mkdirsForFile(String filePath){
		String absPath=filePath;
		String tmpPath=absPath.substring(0,absPath.lastIndexOf(File.separator));
		File tmp=new File(tmpPath);
		if(!tmp.exists()){
			tmp.mkdirs();
		}
	}
	public static void main(String[] args) {
		compressFoldToZip("D:\\new2\\feiq", "D:\\zip\\my.zip");
//		unZipToFold("D:\\zip\\my.zip", "D:\\zip\\unzip");
	}
}


```
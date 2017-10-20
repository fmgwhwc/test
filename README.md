package com.cusc.common.util;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.HttpHeaders;

/**
 * Utility class for HTTP headers creation.
 */
public class HeaderUtil {

    private static final Logger log = LoggerFactory.getLogger(HeaderUtil.class);

    public static HttpHeaders createAlert(String message, String param) {
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-myappApp-alert", message);
        headers.add("X-myappApp-params", param);
        return headers;
    }

    public static HttpHeaders createEntityCreationAlert(String entityName, String param) {
        return createAlert("A new " + entityName + " is created with identifier " + param, param);
    }

    public static HttpHeaders createEntityUpdateAlert(String entityName, String param) {
        return createAlert("A " + entityName + " is updated with identifier " + param, param);
    }

    public static HttpHeaders createEntityDeletionAlert(String entityName, String param) {
        return createAlert("A " + entityName + " is deleted with identifier " + param, param);
    }

    public static HttpHeaders createFailureAlert(String entityName, String errorKey, String defaultMessage) {
        log.error("Entity creation failed, {}", defaultMessage);
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-myappApp-error", defaultMessage);
        headers.add("X-myappApp-params", entityName);
        return headers;
    }
}







package com.cusc.remoteconfig.domain;

import java.util.Collections;
import java.util.List;

public class Page {

	private static final int DEFAULT_PAGESIZE = 20;

	public static final Page DEFAULT = new Page(Integer.MAX_VALUE);
	/**
	 * 每页的记录数量
	 */
	private int pageSize = DEFAULT_PAGESIZE;
	/**
	 * 页码（当前页）
	 */
	private int currentPage;
	/**
	 * 总页数
	 */
	private int totalPage = 0;
	/**
	 * 总记录数
	 */
	private int totalCount = 0;
	/**
	 * 结果集
	 */
	private List<?> result = Collections.emptyList();

	public Page() {
	}

	public Page(int pageSize) {
		this.setPageSize(pageSize);
	}

	public Page(int currentPage, int pageSize) {
		this.setcurrentPage(currentPage);
		this.setPageSize(pageSize);
	}

	public Page(int currentPage, int pageSize, int totalCount, List<?> result) {
		this.setcurrentPage(currentPage);
		this.setPageSize(pageSize);
		this.setTotalCount(totalCount);
		this.setResult(result);
	}

	/**
	 * 获得每页的记录数量，默认为DEFAULT_PAGESIZE
	 */
	public Integer getPageSize() {
		return pageSize;
	}

	/**
	 * 设置每页的记录数量，低于1时自动调整为DEFAULT_PAGESIZE
	 */
	public void setPageSize(int pageSize) {
		this.pageSize = (pageSize < 1) ? DEFAULT_PAGESIZE : pageSize;
	}

	/**
	 * 获取当前页的页号，序号从1开始，默认为1
	 */
	public Integer getcurrentPage() {
		return currentPage;
	}

	/**
	 * 设置当前页的页号，序列从1开始，低于1时自动调整为1
	 */
	public void setcurrentPage(int currentPage) {
		this.currentPage = (currentPage < 1) ? 1 : currentPage;
	}

	public int getTotalCount() {
		return totalCount;
	}

	/**
	 * 设置总记录数，默认值为0
	 */
	public void setTotalCount(int totalCount) {
		this.totalCount = (totalCount < 0) ? 0 : totalCount;
	}

	/**
	 * 总页数
	 */
	public int getTotalPage() {
		totalPage = totalCount / pageSize;
		if (totalCount % pageSize > 0) {
			totalPage++;
		}
		return totalPage;
	}

	public List<?> getResult() {
		return result;
	}

	public void setResult(List<?> result) {
		this.result = result;
	}

	/**
	 * 是否还有上一页
	 */
	public boolean getHasPre() {
		return currentPage - 1 >= 1;
	}

	/**
	 * 是否还有下一页
	 */
	public boolean getHasNext() {
		return currentPage + 1 <= getTotalPage();
	}

	/**
	 * 取得上页的页号，序号从1开始
	 */
	public int getPrePage() {
		if (this.getHasPre()) {
			return currentPage - 1;
		}
		return currentPage;
	}

	/**
	 * 取得下页的页码，序号从1开始
	 */
	public int getNextPage() {
		if (this.getHasNext()) {
			return currentPage + 1;
		}
		return currentPage;
	}

	/***************************************** SQL开始 ****************************/
	/**
	 * 偏移量，用于sql分页查询（从0开始）
	 */
	public int getOffset() {
		return ((currentPage - 1) * pageSize);
	}

	public int getLimit() {
		return pageSize;
	}

	/***************************************** SQL结束 ****************************/

	@Override
	public String toString() {
		StringBuffer sb = new StringBuffer(getClass().getName());
		sb.append("[currentPage=");
		sb.append(this.getcurrentPage());
		sb.append(",pageSize=");
		sb.append(this.getPageSize());
		sb.append(",totalPage=");
		sb.append(this.getTotalPage());
		sb.append(",totalCount=");
		sb.append(this.getTotalCount());
		sb.append(",resultSize=");
		sb.append(this.getResult().size());
		sb.append("]");
		return sb.toString();
	}
}





	
	@ApiOperation(value = "根据条件获取sim卡信息列表", notes = "页面展示该列表")
	@RequestMapping(value = "getSimProfileList", method = RequestMethod.POST, consumes = "application/json")
	public ResponseEntity<Page> getSimProfileList(@RequestBody SimProfile simProfile) {
		Page page = new Page(simProfile.getCurrentPage(), simProfile.getPageSize());
		Page simProfilePage = commonBaseService.getSimProfileList(simProfile, page);
		logger.info("getSimProfileList:获取数据成功");
		return ResponseEntity.ok().headers(HeaderUtil.createAlert("", CodeMessageConstant.SUCCESS_MESSAGE))
				.body(simProfilePage);

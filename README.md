# mnuo.github.io
package com.rt.ssm;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import javax.servlet.http.HttpServletRequest;

import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;
import org.springframework.web.multipart.MultipartFile;

import com.rt.ssm.data.exception.DataException;
import com.rt.ssm.domain.StudyCourse;
import com.rt.ssm.domain.StudyPlanStudyRecordSum;
import com.rt.ssm.model.TStudyCourse;
import com.rt.ssm.model.TStudyLearningNode;
import com.rt.ssm.model.TStudyPlanCourse;
import com.rt.ssm.model.TStudyRecord;
import com.rt.ssm.model.TSysUser;
import com.rt.ssm.model.repository.ICustomCommonRepository;
import com.rt.ssm.resp.Response;
import com.rt.ssm.resp.StudyPlan;
import com.rt.ssm.service.IMessageService;
import com.rt.ssm.service.IStudyService;
import com.rt.ssm.service.ISysService;
import com.rt.ssm.util.BeanUtil;
import com.rt.ssm.util.EntityUtil;
import com.rt.ssm.util.PageUtil;
import com.rt.ssm.util.StringUtils;
import com.rt.ssm.util.WxUtil;

@RestController
public class StudyController {
//	
	public static final String APP_ID = "wx7fac8829271cadfe";
	public static final String APP_SECRET = "23770e60468015ada3a295c57c0c28af"; // 小程序secret
	@Autowired
	IStudyService studyService;
	@Autowired
	ISysService sysService;
	@Autowired
	IMessageService messageService;
	
	@Autowired
	ICustomCommonRepository<?, ?> commonreporitory;
	/**
	 * @param token
	 * @param yyyyMm  
	 * @return
	 */
	@RequestMapping("/study/getStudyPlan")
	public Response getStudyPlan(String token, String yyyyMm, String page, String pageSize) {	
		Response resp = new Response();
		try {
			TSysUser user = sysService.getUserWx(token, "/study/getStudyPlan");
			Map<String, String> params = new HashMap<String, String>();
			params.put("orgId", user.getOrgId());
			params.put("tradeId", user.getTradeId());
			params.put("yyyyMm", yyyyMm);
			params.put("userId", user.getUId());
			PageUtil pageUtil = new PageUtil();
			pageUtil.setPageCount(StringUtils.getStringInteger(page, 1));
			pageUtil.setPageSize(StringUtils.getStringInteger(pageSize, 8));
			System.out.println(pageUtil);
			List<Object[]> list = studyService.getStudyPlanByUser(params, pageUtil);
			List<StudyPlanStudyRecordSum> mlist = EntityUtil.castEntity(list, StudyPlanStudyRecordSum.class);
			resp.setErrFlag("Y");
			resp.setData(mlist);
		} catch (DataException e) {
			resp.setErrFlag("N");
			resp.setErrMsg(e.getMessage());
		} catch (Exception e) {
			resp.setErrFlag("N");
			resp.setErrMsg(e.getMessage());
			e.printStackTrace();
		}
		
		return resp ;
	}
	
	@RequestMapping("/study/getStudyPlanByMonth")
	public Response getStudyPlanByMonth(String token, String yyyyMm) {	
		Response resp = new Response();
		try {
			TSysUser user = sysService.getUserWx(token, "/study/getStudyPlan");
			Map<String, String> params = new HashMap<String, String>();
			params.put("orgId", user.getOrgId());
			params.put("tradeId", user.getTradeId());
			params.put("yyyyMm", yyyyMm);
			params.put("userId", user.getUId());
			List<Object[]> list = studyService.getStudyPlanByUser(params);
			List<StudyPlanStudyRecordSum> mlist = EntityUtil.castEntity(list, StudyPlanStudyRecordSum.class);
			resp.setErrFlag("Y");
			resp.setData(mlist);
		} catch (DataException e) {
			resp.setErrFlag("N");
			resp.setErrMsg(e.getMessage());
		} catch (Exception e) {
			resp.setErrFlag("N");
			resp.setErrMsg(e.getMessage());
			e.printStackTrace();
		}
		
		return resp ;
	}
	
	@RequestMapping("/study/getStudyPlanCourse")
	public Response getStudyPlanCourse(HttpServletRequest request, String token, String planId, String page, String pageSize) {
		Response resp = new Response();
		try {
			TSysUser user = sysService.getUserWx(token, "/study/getStudyPlanCourse");
			Map<String, String> params = new HashMap<String, String>();
			params.put("orgId", user.getOrgId());
			params.put("tradeId", user.getTradeId());
			PageUtil pageUtil = new PageUtil();
			pageUtil.setPageCount(StringUtils.getStringInteger(page, 1));
			pageUtil.setPageSize(StringUtils.getStringInteger(pageSize, 8));
			System.out.println(pageUtil);
//			//获取request上下文
//			String basePath = sysService.getBasePath(request);
//			Map<String, String> map = new HashMap<String, String>();
//			map.put("image", basePath);  //将image属性添加http://..... 前缀用作客户端访问
			List<Object[]> list = studyService.getstudyPlanCourseList1(planId);
			
			List<StudyCourse> tmpList = new ArrayList<StudyCourse>();
			for(Object[] dataArr: list) {
				TStudyCourse course = (TStudyCourse) dataArr[0];
				TStudyPlanCourse pc = (TStudyPlanCourse) dataArr[1];
				StudyCourse target = new StudyCourse();
				BeanUtils.copyProperties(course, target );
				int coutn = studyService.getFinishCount(user.getUId(), planId, course.getUId());
				target.setFinishCount(coutn);
				target.setBatchNum(pc.getBatchNum());
				tmpList.add(target);
			}
//			System.out.println(tmpList);
			resp.setErrFlag("Y");
			resp.setData(tmpList);
		} catch (DataException e) {
			resp.setErrFlag("N");
			resp.setErrMsg(e.getMessage());
		} catch (Exception e) {
			resp.setErrFlag("N");
			resp.setErrMsg(e.getMessage());
			e.printStackTrace();
		}
		
		return resp ;
	}
	
	@RequestMapping("/study/getStudyNodeAndRecord")
	public Response getStudyNodeAndRecord(String token, String planId, String courseId) {
		Response resp = new Response();
		try {
			TSysUser user = sysService.getUserWx(token, "/study/getStudyNodeAndRecord");
			TStudyLearningNode node =this.studyService.getStudyLearningNode(user.getUId(), planId, courseId);
			TStudyRecord record = this.studyService.getStudyRecord(user.getUId(), planId, courseId, 0);
			StudyPlan plan = new StudyPlan();
			plan.setNode(node);
			plan.setRecord(record);
			resp.setErrFlag("Y");
			resp.setData(plan);
		} catch (DataException e) {
			resp.setErrFlag("N");
			resp.setErrMsg(e.getMessage());
		} catch (Exception e) {
			resp.setErrFlag("N");
			resp.setErrMsg(e.getMessage());
			e.printStackTrace();
		}
		
		return resp ;
	}
	
	@RequestMapping("/study/finishStudyCourse")
	public Response finishStudyCourse(String token, String planId, String courseId) {
		Response resp = new Response();
		try {
			TSysUser user = sysService.getUserWx(token,"/study/finishStudyCourse");
			this.studyService.finishStudyCourse(user.getUId(), planId, courseId);
			resp.setErrFlag("Y");
			resp.setData("");
		} catch (DataException e) {
			resp.setErrFlag("N");
			resp.setErrMsg(e.getMessage());
		} catch (Exception e) {
			resp.setErrFlag("N");
			resp.setErrMsg(e.getMessage());
			e.printStackTrace();
		}
		
		return resp ;
	}
	
	@RequestMapping("/study/updateStudyRecord")
	public Response updateStudyRecord(String token, String planId, String courseId, String recordTimeLine) {
		Response resp = new Response();
		try {
			TSysUser user = sysService.getUserWx(token, "/study/updateStudyRecord");
			this.studyService.updateStudyRecord(user.getUId(), planId, courseId, recordTimeLine);
			
			resp.setErrFlag("Y");
		} catch (DataException e) {
			resp.setErrFlag("N");
			resp.setErrMsg(e.getMessage());
		} catch (Exception e) {
			resp.setErrFlag("N");
			resp.setErrMsg(e.getMessage());
			e.printStackTrace();
		}
		
		return resp ;
	}
	
	
	@RequestMapping("/study/uploadImage")
	public Response uploadImage(@RequestParam("file") MultipartFile file, String token, String planId, String courseId) {
		Response resp = new Response();
		try {
			TSysUser user = sysService.getUserWx(token, "/study/uploadImage");
			studyService.saveUserImage(file, user.getUId(), planId, courseId);
//			List<TSysImage> imageList = studyService.getUserImageList(user.getUId(), planId, courseId);
			resp.setErrFlag("Y");
			resp.setData("");
		} catch (DataException e) {
			resp.setErrFlag("N");
			resp.setErrMsg(e.getMessage());
		} catch (Exception e) {
			resp.setErrFlag("N");
			resp.setErrMsg(e.getMessage());
			e.printStackTrace();
		}
		
		return resp ;
    }
	
	@RequestMapping("/study/getUserImageList")
	public Response getUserImageList(String token, String planId, String courseId) {
		Response resp = new Response();
		try {
			TSysUser user = sysService.getUserWx(token, "/study/getUserImageList");
			System.out.println(user.getUId() + "," + planId + "," + courseId);
			String[] imageIds = studyService.getUserImageUrlList(user.getUId(), planId, courseId, 4);
			resp.setErrFlag("Y");
			resp.setData(imageIds);
		} catch (DataException e) {
			resp.setErrFlag("N");
			resp.setErrMsg(e.getMessage());
		} catch (Exception e) {
			resp.setErrFlag("N");
			resp.setErrMsg(e.getMessage());
			System.out.println(e.getMessage());
		}
		
		return resp ;
    }
	
	@RequestMapping("/wxLogin")
	public Response wxLogin(String code) {
		Response resp = new Response();
		try {
			String url = "https://api.weixin.qq.com/sns/jscode2session?appid=" + APP_ID + "&secret=" + APP_SECRET + "&js_code=" + code + "&grant_type=authorization_code";
			RestTemplate rest = new RestTemplate();
			String json = rest.getForEntity(url, String.class).getBody();  
			System.out.println(json);
			
			resp.setErrFlag("Y");
			resp.setData(json);
		} catch (Exception e) {
			resp.setErrFlag("N");
			resp.setErrMsg(e.getMessage());
			e.printStackTrace();
		} finally {
			
		}
		
		return resp;
	}
	
	@RequestMapping("/weChatlogin")
	public Response weChatlogin(String token) {
		Response resp = new Response();
		try {
			String phoneNo = WxUtil.getPhoneNo(token);
			System.out.println(phoneNo);
			Map m = new HashMap();
			m.put("phoneNo", phoneNo);
			
			resp.setErrFlag("Y");
			resp.setData(m);
		} catch (Exception e) {
			resp.setErrFlag("N");
			resp.setErrMsg(e.getMessage());
			e.printStackTrace();
		} finally {
			
		}
		
		return resp;
	}
	
	
}
package com.rt.ssm;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import com.rt.ssm.data.exception.DataException;
import com.rt.ssm.model.TSysNotice;
import com.rt.ssm.model.TSysPublicMessage;
import com.rt.ssm.model.TSysUser;
import com.rt.ssm.model.repository.ICustomCommonRepository;
import com.rt.ssm.resp.Response;
import com.rt.ssm.service.IMessageService;
import com.rt.ssm.service.IStudyService;
import com.rt.ssm.service.ISysService;
import com.rt.ssm.util.PageUtil;
import com.rt.ssm.util.StringUtils;

@RestController
public class MessageController {
	@Autowired
	IStudyService studyService;
	@Autowired
	ISysService sysService;
	@Autowired
	IMessageService messageService;
	
	@Autowired
	ICustomCommonRepository<?, ?> commonreporitory;
	
	
	@RequestMapping("/study/getNoticeList")
	public Response getNoticeList(String token, String page, String pageSize) {
		Response resp = new Response();
		try {
			TSysUser user = sysService.getUserWx(token, "/study/getNoticeList");
			PageUtil pageUtil = new PageUtil();
			pageUtil.setPageCount(StringUtils.getStringInteger(page, 1));
			pageUtil.setPageSize(StringUtils.getStringInteger(pageSize, 8));
			List<TSysNotice> list = messageService.getSysNotice(user.getUId(), pageUtil);
			for(TSysNotice notice: list) {
				notice.setReadTimeLong(notice.getReadTime() != null ? notice.getReadTime().getTime() : 0);
				notice.setPublishTimeLong(notice.getPublishTime().getTime());
			}
			
			resp.setErrFlag("Y");
			resp.setData(list);
		} catch (DataException e) {
			resp.setErrFlag("N");
			resp.setErrMsg(e.getMessage());
		} catch (Exception e) {
			resp.setErrFlag("N");
			resp.setErrMsg(e.getMessage());
			e.printStackTrace();
		}
		
		return resp ;
	}
	
	@RequestMapping("/study/getPublicMessageList")
	public Response getPublicMessageList(String token, String page, String pageSize) {
		Response resp = new Response();
		try {
			TSysUser user = sysService.getUserWx(token, "/study/getPublicMessageList");
			PageUtil pageUtil = new PageUtil();
			pageUtil.setPageCount(StringUtils.getStringInteger(page, 1));
			pageUtil.setPageSize(StringUtils.getStringInteger(pageSize, 8));
			List<TSysPublicMessage> list = messageService.getSysPublicMessage(pageUtil, user.getUId());
			for(TSysPublicMessage notice: list) {
				notice.setPublishTimeLong(notice.getPublishTime().getTime());
			}
			resp.setErrFlag("Y");
			resp.setData(list);
		} catch (DataException e) {
			resp.setErrFlag("N");
			resp.setErrMsg(e.getMessage());
		} catch (Exception e) {
			resp.setErrFlag("N");
			resp.setErrMsg(e.getMessage());
			e.printStackTrace();
		}
		
		return resp ;
	}
	
	/**
	 * 获取未读总数
	 * @param token
	 * @param page
	 * @param pageSize
	 * @return
	 */
	@RequestMapping("/study/getMsgUnReadTotalCount")
	public Response getMsgUnReadTotalCount(String token, String page, String pageSize) {
		Response resp = new Response();
		try {
			TSysUser user = sysService.getUserWx(token,"/study/getMsgUnReadTotalCount");
			Long totalCount = messageService.getMsgUnReadTotalCount(user.getUId());
			resp.setErrFlag("Y");
			resp.setData(totalCount);
		} catch (DataException e) {
			resp.setErrFlag("N");
			resp.setErrMsg(e.getMessage());
		} catch (Exception e) {
			resp.setErrFlag("N");
			resp.setErrMsg(e.getMessage());
			e.printStackTrace();
		}
		
		return resp ;
	}
	
	/**
	 * @param token
	 * @param page
	 * @param pageSize
	 * @return
	 */
	@RequestMapping("/study/readPublicMessageContent")
	public Response readPublicMessageContent(String token, String msgId) {
		System.out.println("/study/readPublicMessageContent");
		Response resp = new Response();
		try {
			TSysUser user = sysService.getUserWx(token, "/study/readPublicMessageContent");
			messageService.readPublicMessage(user.getUId(), msgId);
			resp.setErrFlag("Y");
			resp.setData("");
		} catch (DataException e) {
			resp.setErrFlag("N");
			resp.setErrMsg(e.getMessage());
		} catch (Exception e) {
			resp.setErrFlag("N");
			resp.setErrMsg(e.getMessage());
			e.printStackTrace();
		}
		
		return resp ;
	}
	
	/**
	 * @param token
	 * @param page
	 * @param pageSize
	 * @return
	 */
	@RequestMapping("/study/readNoticeContent")
	public Response readNoticeContent(String token, String msgId) {
		System.out.println("/study/readNoticeContent");
		Response resp = new Response();
		try {
			TSysUser user = sysService.getUserWx(token,"/study/readNoticeContent");
			messageService.readNoticeMessage(user.getUId(), msgId);
			resp.setErrFlag("Y");
			resp.setData("");
		} catch (DataException e) {
			resp.setErrFlag("N");
			resp.setErrMsg(e.getMessage());
		} catch (Exception e) {
			resp.setErrFlag("N");
			resp.setErrMsg(e.getMessage());
			e.printStackTrace();
		}
		
		return resp ;
	}
	
	
	
	
}

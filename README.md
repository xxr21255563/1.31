# 1.31from manim import *
import numpy as np

class SwordDance(Scene):
    def construct(self):
        # 加载SVG人物素材（需自行准备或使用示例路径）
        swordsman = SVGMobject("woman_swordsman.svg").scale(1.5).shift(DOWN*0.5)
        sword = SVGMobject("sword.svg").scale(0.8).next_to(swordsman.get_right(), RIGHT)
        
        # 调整剑的初始位置
        sword.rotate(-45*DEGREES).shift(UP*0.3 + LEFT*0.2)
        
        # 创建剑光轨迹
        def create_sword_trail():
            return TracedPath(sword.get_center, 
                            stroke_color=GOLD_E,
                            stroke_width=8,
                            dissipating_time=0.3)

        # 创建剑气粒子
        particles = VGroup()
        for _ in range(30):
            p = Dot(color=random_bright_color(), radius=0.05)
            p.velocity = np.array([(np.random.rand()-0.5)*3, 
                                 (np.random.rand()-0.5)*3, 
                                 0])
            particles.add(p)
        
        # 主动画序列
        self.play(
            FadeIn(swordsman, shift=UP),
            Create(sword, run_time=1.5),
        )
        
        trail = create_sword_trail()
        self.add(trail, particles)
        
        # 剑招动作组合
        sword_actions = [
            {"angle": 120, "direction": RIGHT, "run_time": 0.8},
            {"angle": -90, "direction": LEFT, "run_time": 1.2},
            {"angle": 180, "direction": UR, "run_time": 1},
            {"angle": -150, "direction": DL, "run_time": 0.6}
        ]
        
        # 粒子更新函数
        def update_particles(mob, dt):
            for p in mob:
                p.shift(p.velocity * dt)
                p.velocity += np.array([0, -1, 0]) * dt  # 重力模拟
                p.set_opacity(p.get_opacity()*0.92)
        
        particles.add_updater(update_particles)
        
        # 执行剑术动作
        for action in sword_actions:
            self.play(
                Rotate(sword, 
                      angle=action["angle"]*DEGREES,
                      about_point=swordsman.get_right(),
                      rate_func=there_and_back),
                swordsman.animate.shift(action["direction"]*0.3),
                run_time=action["run_time"]
            )
        
        # 收剑入鞘动画
        sheath = Rectangle(height=1.2, width=0.3, 
                          fill_color=GREY, fill_opacity=1,
                          stroke_width=0).next_to(swordsman.get_left(), LEFT)
        
        self.play(
            Rotate(sword, 45*DEGREES, about_point=swordsman.get_hand()),
            sword.animate.move_to(sheath.get_center()),
            FadeIn(sheath, shift=LEFT),
            run_time=1.5
        )
        
        self.wait(2)

# 运行命令（需安装manim）：
# manim -pql sword_dance.py SwordDance --format=mp4
